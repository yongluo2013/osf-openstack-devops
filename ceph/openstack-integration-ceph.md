##openstack integration ceph

在ceph 中创建4个pool
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

拷贝对应证书到节点

	scp /etc/ceph/ceph.client.images.keyring root@10.0.0.10:/etc/ceph/ceph.client.images.keyring
	scp /etc/ceph/ceph.conf root@10.0.0.10:/etc/ceph/ceph.conf

修改glance 配置文件

	openstack-config --set /etc/glance/glance-api.conf DEFAULT default_store rbd
	openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True

	openstack-config --set /etc/glance/glance-api.conf glance_store stores rbd
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_pool images
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_user images
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_ceph_conf  /etc/ceph/ceph.conf
	openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_chunk_size  8

	chown glance:glance -R /etc/glance

	 systemctl restart openstack-glance-api

转换镜像格式为raw格式

	qemu-img convert -f qcow2 -O raw cirros-0.3.3-x86_64-disk.img cirros-0.3.3-x86_64-disk.raw

上传镜像

	glance image-create --name "cirros-0.3.3-x86_64" --file cirros-0.3.3-x86_64-disk.raw --disk-format raw --container-format bare --is-public True --progress

##配置nova 使用ceph


	cat > secret.xml <<EOF
	<secret ephemeral='no' private='no'>
	  <uuid>8d204fef-e606-49bf-921c-c1566e2e784d</uuid>
	  <usage type='ceph'>
	    <name>client.volumes secret</name>
	  </usage>
	</secret>
	EOF

	sudo virsh secret-define --file secret.xml

	sudo virsh secret-set-value --secret 8d204fef-e606-49bf-921c-c1566e2e784d --base64 $(cat client.volumes.key) && rm client.volumes.key secret.xml


配置nova.conf


openstack-config --set /etc/nova/nova.conf libvirt images_type rbd
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_pool volumes
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/nova/nova.conf libvirt inject_password false
openstack-config --set /etc/nova/nova.conf libvirt inject_key false
openstack-config --set /etc/nova/nova.conf libvirt inject_partition -2
openstack-config --set /etc/nova/nova.conf libvirt rbd_user volumes
openstack-config --set /etc/nova/nova.conf libvirt rbd_secret_uuid 8d204fef-e606-49bf-921c-c1566e2e784d
openstack-config --set /etc/nova/nova.conf libvirt live_migration_flag "VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST"



##配置cinder 使用ceph

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




