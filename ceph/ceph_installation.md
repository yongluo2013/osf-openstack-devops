#Ceph installation

网络拓扑与节点

	10.0.0.110 admin
	10.0.0.50  mon0
	10.0.0.60 osd0
	10.0.0.61 osd1

所有节点配置ceph yum 源

	sudo vim /etc/yum.repos.d/ceph.repo

	[ceph]
	name=Ceph noarch packages
	baseurl=http://ceph.com/rpm-giant/el7/x86_64
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=http://ceph.com/rpm-giant/el7/noarch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

安装deploy工具

	sudo yum update && sudo yum install ceph-deploy

安装ntp 

	sudo yum install ntp ntpdate ntp-doc


配置所有ceph 节点支持ssh key 登录

	ssh  ceph@ceph_mon0
	sudo useradd -d /home/ ceph -m  ceph
	sudo passwd  ceph
	echo " ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ ceph
	sudo chmod 0440 /etc/sudoers.d/ ceph

生成SSH keys

	ssh-keygen

	Generating public/private key pair.
	Enter file in which to save the key (/ceph-admin/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /ceph-admin/.ssh/id_rsa.
	Your public key has been saved in /ceph-admin/.ssh/id_rsa.pub.

支持ssh-copy

	ssh-copy-id  ceph@ceph_mon0
	ssh-copy-id  ceph@ceph_osd0
	ssh-copy-id  ceph@ceph_osd1

所有节点禁用selinux 

	vi /ect/sysconfig/selinux

	SELINUX=disable

所有节点修改hostname

	vi /etc/hosts

	10.0.0.50  mon0
	10.0.0.60 osd0
	10.0.0.61 osd1

admin 节点上执行

	mkdir my-cluster
	cd my-cluster

创建一个ceph cluster 的配置文件目录

	ceph-deploy new mon0

批量安装ceph 节点的rpm包

	ceph-deploy install mon0 osd0 osd1 --no-adjust-repos 

###初始化monitor 节点配置

	ceph-deploy mon create-initial

###启用osd

	ssh osd0
	sudo mkdir /var/local/osd0
	exit

	ssh osd1
	sudo mkdir /var/local/osd1
	exit

	ceph-deploy osd prepare osd0:/var/local/osd0 osd1:/var/local/osd1

	ceph-deploy osd activate osd0:/var/local/osd0 osd1:/var/local/osd1

	ceph-deploy admin mon0 osd0 osd1


Once you have added your new Ceph Monitors, Ceph will begin synchronizing the monitors and form a quorum. You can check the quorum status by executing the following:

	ceph quorum_status --format json-pretty

参考：http://docs.ceph.com/docs/master/start/quick-ceph-deploy/

### 测试object 存储


先查看默认存在的pool,rbd 和 data
	ceph osd lspools

准备一个数据文件
	echo "hello my data!" > testfile.txt
把testfile.txt 放到data pool 中

	rados put test-object-1 testfile.txt --pool=data

查看pool 中的数据

	rados -p data ls

参考数据文件的存储分布

	ceph osd map data test-object-1
删除对象文件

	rados rm test-object-1 --pool=data

参考：http://docs.ceph.com/docs/master/start/quick-ceph-deploy/



## 测试Linux 直接使用ceph 块存储


centos7 默认kernel没有rbd 支持，安装ceph fs kernel 支持，需要重新编译或者直接rpm 方式升级kernel，重启Linux。

先安装源

	yum install http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm 

	yum --enablerepo=elrepo-kernel install kernel-ml

	reboot

运行如下命令不报错，说明rbd module 已经安装。

	modprobe rbd

先创建一个块

	rbd create block1-ceph-client1 --size 200

映射到本地设备
	rbd map block1-ceph-client1

显示有哪些映射设备
	rbd showmapped

格式化块设备

	mkfs.ext4 /dev/rbd/rbd/block1-ceph-client1
	mount /dev/rbd/rbd/block1-ceph-client1 /mnt

###测试libvirt VM 使用ceph 块存储

安装kvm和libvirt

	yum -y install qemu-kvm libvirt virt-install bridge-utils


先检查qemu-kvm 是否支持rbd，输出为空说明不支持

	qemu-img | grep "Supported format"|grep rbd 

由于qemu-kvm 在centos7 目前依旧默认不支持rbd，需要重新编译rbd 


安装yum-utils，用于编译源码

	yum groupinstall -y "Development Tools"
	yum install -y yum-utils rpm-build

重centos 源仓库下载源码rpm 到本地

	yumdownloader --source qemu-kvm
	mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
	echo '%_topdir %(echo $HOME)/rpmbuild' > ~/.rpmmacros

解压安装源码包

	rpm -ivh qemu-kvm-*
	cd ~/rpmbuild/SPECS
	
	vi qemu-kvm.spec

修改默认spec ，找到 --block-drv-rw-whitelist 这一项后边添加rbd 参数

	rpmbuild -bb qemu-kvm.spec 

如果遇到依赖问题，则用yum 直接安装依赖即可， 完成编译后，在  ../RPMS/x86_64/目录下就生成了rpm包


卸载原来的rpm

	rpm -e --nodeps qemu-img
	rpm -e --nodeps qemu-kvm
	rpm -e --nodeps qemu-kvm-tools

安装新的rpm

	rpm -Uvh qemu-*

参考：

	http://vault.centos.org/7.0.1406/updates/Source/SPackages/
	http://crashcourse.ca/content/working-source-rpms-under-centos

测试创建一个给vm 使用的rdb

	ceph osd pool create libvirt-pool 128 128

	ceph osd lspools

	ceph auth get-or-create client.libvirt mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=libvirt-pool'

	ceph auth list

	qemu-img create -f rbd rbd:libvirt-pool/new-libvirt-image 2G

	rbd -p libvirt-pool ls

生成libvirt secret 文件定义xml 文件

	cat > secret.xml <<EOF
	<secret ephemeral='no' private='no'>
	        <usage type='ceph'>
	                <name>client.libvirt secret</name>
	        </usage>
	</secret>
	EOF

在libvirt 中生成domain，记录uuid为后续使用

	sudo virsh secret-define --file secret.xml
	3a7e333a-649d-472e-8860-824c170bd75e

	ceph auth get-key client.libvirt | sudo tee client.libvirt.key

设置secret 的值
	sudo virsh secret-set-value --secret 3a7e333a-649d-472e-8860-824c170bd75e --base64 $(cat client.libvirt.key) && rm client.libvirt.key secret.xml

创建一个vm （ceph-client.xml），并添加一个新的网络磁盘设备

	<disk type='network' device='disk'>
	        <source protocol='rbd' name='libvirt-pool/new-libvirt-image'>
	                <host name='mon0' port='6789'/>
	        </source>
	        <auth username='libvirt'>
	        	<secret type='ceph' uuid='3a7e333a-649d-472e-8860-824c170bd75e'/>
			</auth>
	        <target dev='vda' bus='virtio'/>
	</disk>

启动vm

	virsh define ceph-client.xml
	virsh start ceph-client

登录vm 查看是否出现新的磁盘设备


	sudo virsh list

Check to see if the VM is communicating with Ceph. Replace {vm-domain-name} with the name of your VM domain:

	sudo virsh qemu-monitor-command --hmp ceph-client 'info block'

Check to see if the device from <target dev='hdb' bus='ide'/> appears under /dev or under proc/partitions.

	ls dev
	cat proc/partitions

参考文档：http://ceph.com/docs/master/rbd/libvirt/


相关问题：
http://my.oschina.net/fkkeee/blog/317127 （中文安装手册）
https://ask.openstack.org/en/question/56148/rdo-icehouse-with-ceph-libvirt-error/ 
http://blog.mit.bme.hu/meszaros/en/node/190 
http://www.linuxqq.net/archives/1247.html