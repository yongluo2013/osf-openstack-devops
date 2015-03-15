## OS installation on HP G8

### Device information

	https://10.170.5.25/   embms/embms1234

### Disk Array Configuration

Set up all of disk with RAID0

### OS disk partition

all of fs use ext4

	/boot  300mb     
	/      30G
	sweep  30G
	/var   all of left disk splace


### IP address planning

The floating IP block 10.175.183.0/24 , OAM network 10.170.5.0/24

	eno1: 192.168.1.0/24 pxe network
	eno2: 192.168.2/0/24 management network
	eno3: 192.168.3.0/24 internal/private network
	eno4: 192.168.4.0/24 storage network
	eno5: 10.175.183.0/24 external/public network

config.cnsheta.com
	eno1: 192.168.1.111
	eno2: 192.168.2.111
	eno3: 192.168.3.111
	eno4: 192.168.4.111
	eno5: 10.175.183.111 

controller0.cnsheta.com
	eno1: 192.168.1.112
	eno2: 192.168.2.112
	eno3: 192.168.3.112
	eno4: 192.168.4.112
	eno5: 10.175.183.112 

controller1.cnsheta.com
	eno1: 192.168.1.113
	eno2: 192.168.2.113
	eno3: 192.168.3.113
	eno4: 192.168.4.113
	eno5: 10.175.183.113

network0.cnsheta.com
	eno1: 192.168.1.114
	eno2: 192.168.2.114
	eno3: 192.168.3.114
	eno4: 192.168.4.114
	eno5: 10.175.183.114

network1.cnsheta.com
	eno1: 192.168.1.115
	eno2: 192.168.2.115
	eno3: 192.168.3.115
	eno4: 192.168.4.115
	eno5: 10.175.183.115

compute0.cnsheta.com
	eno1: 192.168.1.10
	eno2: 192.168.2.10
	eno3: 192.168.3.10
	eno4: 192.168.4.10
	eno5: 10.175.183.10

compute1.cnsheta.com
	eno1: 192.168.1.11
	eno2: 192.168.2.11
	eno3: 192.168.3.11
	eno4: 192.168.4.11
	eno5: 10.175.183.11

	

### yum repo server installation

### Puppet & Foreman installation

### Ceph installation 

先准备为ceph 准备repo

[ceph]
name=Ceph noarch packages
baseurl=http://192.168.1.114/rpm-giant/el7/x86_64
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=http://192.168.1.114/ceph_release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=http://192.168.1.114/rpm-giant/el7/noarch
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=http://192.168.1.114/ceph_release.asc


admin 节点上执行(config)

	mkdir my-cluster
	cd my-cluster

创建一个ceph cluster 的配置文件目录

	ceph-deploy new compute0

批量安装ceph 节点的rpm包,注意--no-adjust-repos  参数，不需要额外自动添加新的repo

	ceph-deploy install compute0 compute1 compute2 --no-adjust-repos 

###初始化monitor 节点配置

	ceph-deploy mon create-initial

###启用osd

	ssh compute1
	sudo mkdir /var/local/osd1
	exit

	ssh compute2
	sudo mkdir /var/local/osd2
	exit

	ceph-deploy osd prepare compute1:/var/local/osd1 compute2:/var/local/osd2

	ceph-deploy osd activate compute1:/var/local/osd1 compute2:/var/local/osd2

	ceph-deploy admin compute0 compute1 compute2


注意开启防火墙

	-A INPUT -p tcp -m multiport --ports 6789 -m comment --comment "6789 - mon" -m state --state NEW -j ACCEPT
	-A INPUT -p tcp -m multiport --ports 6800 -m comment --comment "6800 - osd0" -m state --state NEW -j ACCEPT
	-A INPUT -p tcp -m multiport --ports 6801 -m comment --comment "6801 - osd1" -m state --state NEW -j ACCEPT
	-A INPUT -p tcp -m multiport --ports 6802 -m comment --comment "6802 - osd2" -m state --state NEW -j ACCEPT
	-A INPUT -p tcp -m multiport --ports 6803 -m comment --comment "6803 - osd3" -m state --state NEW -j ACCEPT



在ceph 中创建4个pool,在compute0 创建对应的pool

	ceph osd pool create volumes 128
	ceph osd pool create images 128
	ceph osd pool create vms 128
	ceph osd pool create backups 128

为images 和 user 生成keyring

   sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.images.keyring
   sudo chmod +r /etc/ceph/ceph.client.images.keyring
   sudo ceph-authtool /etc/ceph/ceph.client.images.keyring -n client.images --gen-key
   sudo ceph-authtool -n client.images --cap mon 'allow r' --cap osd 'allow class-read object_prefix rbd_children, allow rwx pool=images' /etc/ceph/ceph.client.images.keyring
   ceph auth add client.images -i /etc/ceph/ceph.client.images.keyring

为volumes创建keyring 和 user 生成keyring
   sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.volumes.keyring
   sudo chmod +r /etc/ceph/ceph.client.volumes.keyring
   sudo ceph-authtool /etc/ceph/ceph.client.volumes.keyring -n client.volumes --gen-key
   sudo ceph-authtool -n client.volumes --cap mon 'allow r' --cap osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rx pool=images' /etc/ceph/ceph.client.volumes.keyring
   ceph auth add client.volumes -i /etc/ceph/ceph.client.volumes.keyring

为backups创建keyring 和 user 生成keyring
   sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.backups.keyring
   sudo chmod +r /etc/ceph/ceph.client.backups.keyring
   sudo ceph-authtool /etc/ceph/ceph.client.backups.keyring -n client.backups --gen-key
   sudo ceph-authtool -n client.backups --cap mon 'allow r' --cap osd 'allow class-read object_prefix rbd_children, allow rwx pool=backups' /etc/ceph/ceph.client.backups.keyring
   ceph auth add client.backups -i /etc/ceph/ceph.client.backups.keyring


修改ceph.conf 文件，添加如下内容

	[client.images]
	keyring = /etc/ceph/ceph.client.images.keyring

	[client.volumes]
	keyring = /etc/ceph/ceph.client.volumes.keyring

	[client.backups]
	keyring = /etc/ceph/ceph.client.backups.keyring

拷贝对应证书到节点，controller 节点需要glance访问mon，其他计算节点需要直接使用osd rbd 块存储。

	scp /etc/ceph/ceph.client.*.keyring root@controller1:/etc/ceph/ceph.client.*.keyring
	scp /etc/ceph/ceph.conf root@controller1:/etc/ceph/ceph.conf

修改controler 节点 ，glance 配置文件

	openstack-config --set /etc/glance/glance-api.conf DEFAULT default_store rbd
	openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True

	openstack-config --set /etc/glance/glance-api.conf glance_store stores rbd
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_pool images
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_user images
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_ceph_conf  /etc/ceph/ceph.conf
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_chunk_size  8

	chown glance:glance -R /etc/glance

	 systemctl restart openstack-glance-api



转换镜像格式为raw格式，rdb 只支持裸格式的磁盘

	qemu-img convert -f qcow2 -O raw cirros-0.3.1-x86_64-disk.img cirros-0.3.1-x86_64-disk.raw

上传镜像

	glance image-create --name "cirros-0.3.1-x86_64" --file cirros-0.3.1-x86_64-disk.raw --disk-format raw --container-format bare --is-public True --progress


转换镜像格式为raw格式

	qemu-img convert -f qcow2 -O raw cirros-0.3.1-x86_64-disk.img cirros-0.3.1-x86_64-disk.raw

上传镜像

	glance image-create --name "cirros-0.3.1-x86_64" --file cirros-0.3.1-x86_64-disk.raw --disk-format raw --container-format bare --is-public True --progress

测试下载

	glance image-download <image-uudi> --file test.img



##nova 支持直接使用ceph 块

各个计算节点安装ceph rpm 包

	sudo yum install python-ceph ceph

检查qemu-kvm 是否支持rbd

	qemu-img | grep "Supported format"|grep rbd 

不支持请update qemu-kvm 包

	yum install -y qemu-kvm* && yum upgrade -y qemu-kvm*


再compute0 节点上(mon0 节点)

	ceph auth get-key client.volumes|ssh compute1 tee client.volumes.key

再各个计算节点上执行

cat > secret.xml <<EOF
	<secret ephemeral='no' private='no'>
	  <uuid>8d204fef-e606-49bf-921c-c1566e2e784d</uuid>
	  <usage type='ceph'>
	    <name>client.volumes secret</name>
	  </usage>
	</secret>
	EOF

sudo virsh secret-define --file secret.xml

sudo virsh secret-set-value --secret 8d204fef-e606-49bf-921c-c1566e2e784d --base64 $(cat client.volumes.key)

配置nova rbd 块

先要kill puppet 守候进程

ps -ef|grep puppet
kill -9 3092

openstack-config --set /etc/nova/nova.conf libvirt images_type rbd
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_pool volumes
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/nova/nova.conf libvirt inject_password false
openstack-config --set /etc/nova/nova.conf libvirt inject_key false
openstack-config --set /etc/nova/nova.conf libvirt inject_partition -2
openstack-config --set /etc/nova/nova.conf libvirt rbd_user volumes
openstack-config --set /etc/nova/nova.conf libvirt rbd_secret_uuid 8d204fef-e606-49bf-921c-c1566e2e784d
openstack-config --set /etc/nova/nova.conf libvirt live_migration_flag "VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST"


重启nova compute 相关服务

	systemctl restart openvswitch libvirtd.service neutron-openvswitch-agent.service openstack-nova-compute.service


##配置cinder 存储节点使用ceph

在volume 节点安装ceph

	sudo yum install -y ceph   python-ceph

	openstack-config --set /etc/cinder/cinder.conf DEFAULT volume_driver cinder.volume.drivers.rbd.RBDDriver
	openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_user volumes
	openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_secret_uuid 8d204fef-e606-49bf-921c-c1566e2e784d
	openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_pool volumes
	openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_ceph_conf /etc/ceph/ceph.conf
	openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_flatten_volume_from_snapshot false
	openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_max_clone_depth 5

从启cinder-volume

	systemctl enable openstack-cinder-volume.service
	systemctl start openstack-cinder-volume.service

创建一个vlume1测试看看是否使用ceph 存储



##扩展一个新的计算节点

vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.111 config.cnsheta.com
192.168.2.113 controller1.cnsheta.com controller1
192.168.2.10 compute0.cnsheta.com compute0
192.168.2.11 compute1.cnsheta.com compute1
192.168.2.12 compute2.cnsheta.com compute2
192.168.2.13 compute3.cnsheta.com compute3
192.168.2.14 compute4.cnsheta.com compute4
192.168.2.15 compute5.cnsheta.com compute5
192.168.2.16 compute6.cnsheta.com compute6
192.168.2.17 compute7.cnsheta.com compute7
192.168.2.18 compute8.cnsheta.com compute8
192.168.2.19 compute9.cnsheta.com compute9
192.168.2.20 compute10.cnsheta.com compute10

vi /etc/yum.repos.d/ceph.repo

[ceph]
name=Ceph noarch packages
baseurl=http://192.168.1.114/rpm-giant/el7/x86_64
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=http://192.168.1.114/ceph_release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=http://192.168.1.114/rpm-giant/el7/noarch
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=http://192.168.1.114/ceph_release.asc

[rdb-qemu]
name=rdb-qemu
baseurl=http://192.168.1.114/qemu
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

安装 ceph 

yum clean all&& yum makecache

sudo yum install -y python-ceph ceph


检查qemu-kvm 是否支持rbd

	qemu-img | grep "Supported format"|grep rbd 

不支持请update qemu-kvm 包

	yum install -y qemu-kvm* && yum upgrade -y qemu-kvm*

拷贝ceph 配置文件

	scp /etc/ceph/ceph.client.*.keyring root@<compute_node>:/etc/ceph/
	scp /etc/ceph/ceph.conf root@<compute_node>:/etc/ceph/


在monitor节点上(mon0 节点) 拷贝key 到计算节点

	ceph auth get-key client.volumes|ssh <compute_node> tee client.volumes.key

在计算节点上导入libvirt secret key

cat > secret.xml <<EOF
	<secret ephemeral='no' private='no'>
	  <uuid>8d204fef-e606-49bf-921c-c1566e2e784d</uuid>
	  <usage type='ceph'>
	    <name>client.volumes secret</name>
	  </usage>
	</secret>
	EOF

sudo virsh secret-define --file secret.xml

sudo virsh secret-set-value --secret 8d204fef-e606-49bf-921c-c1566e2e784d --base64 $(cat client.volumes.key)

配置nova rbd 块

先要kill puppet 守候进程

ps -ef|grep puppet
kill -9 3092
systemctl disable puppet

openstack-config --set /etc/nova/nova.conf libvirt images_type rbd
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_pool volumes
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/nova/nova.conf libvirt inject_password false
openstack-config --set /etc/nova/nova.conf libvirt inject_key false
openstack-config --set /etc/nova/nova.conf libvirt inject_partition -2
openstack-config --set /etc/nova/nova.conf libvirt rbd_user volumes
openstack-config --set /etc/nova/nova.conf libvirt rbd_secret_uuid 8d204fef-e606-49bf-921c-c1566e2e784d
openstack-config --set /etc/nova/nova.conf libvirt live_migration_flag "VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST"


重启nova compute 相关服务

	systemctl restart  openstack-nova-compute.service

## 问题

删除不用的compute node




## 添加新的ceph 节点

ssh-copy-id  root@compute4

ssh root@compute4 "mkdir /var/local/osd4"

ceph-deploy --overwrite-conf osd prepare compute4:/var/local/osd4

ceph-deploy osd activate compute4:/var/local/osd4

ceph-deploy admin compute4

检查状态





##其他问题

###glance-registry  默认没有配置，不能启动问题


1.缺少 glance-registry-paste.ini 文件，下载该文件到/etc/glance目录

wget https://raw.githubusercontent.com/openstack/glance/stable/juno/etc/glance-registry-paste.ini 

2.缺少glance-register 配置

	openstack-config --set /etc/glance/glance-registry.conf DEFAULT debug True
	openstack-config --set /etc/glance/glance-registry.conf DEFAULT verbose True
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://192.168.2.113:5000
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_host 192.168.2.113
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_port 35357
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_protocol http
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken admin_tenant_name services
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken admin_user glance
	openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken admin_password na-mu-va
	openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone

systemctl restart openstack-glance-registry

###mariadb 超过最大连接数问题

	systemctl restart mariadb


直接修改/etc/my.cnf  [mysqld] 中的 max_connections=1000 重启无效

修改 /etc/my.cnf.d/server.cnf


### dashboard 分配floating ip 没有port 显示

参考bug https://bugs.launchpad.net/horizon/+bug/1394051


 






















