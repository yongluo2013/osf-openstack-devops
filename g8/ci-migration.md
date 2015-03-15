### BoLTE CI 迁移

1. user and tenants 创建

user ：ci
tenants ：ci common bmc_15a bmc_16a bdc_15a bdc_16a

keystone tenant-create --name=bdc_14a --description="use to bdc 14a" 
keystone tenant-create --name=bdc_15a --description="use to bdc 15a" 
keystone tenant-create --name=bdc_16a --description="use to bdc 16a" 

keystone tenant-create --name=bmc_15a --description="use to bmc 15a" 
keystone tenant-create --name=bmc_16a --description="use to bmc 16a" 


keystone tenant-create --name=eta --description="use to eta team" 

2.导入镜像

Jenkins slave image

glance image-create --name "jenkins_slave" --file jenkins_slave.raw --disk-format raw --container-format bare --is-public True --progress

cdn slave 14a 15a 16a 

glance image-create --name "15a_cdn" --file 15a_cdn.raw --disk-format raw --container-format bare --is-public True --progress

nfs
yum repo server

16a bdc：
	rhel_bmsc_base_15b
	sles_test_agent_15b

glance image-create --name "rhel_bmsc_base_15b" --file rhel_bmsc_base_15b.raw --disk-format raw --container-format bare --is-public True --progress

glance image-create --name "sles_test_agent_15b" --file sles_test_agent_15b.raw --disk-format raw --container-format bare --is-public True --progress

16a bmc

	rhel6.5-x86_64

glance image-create --name "rhel6.5-x86_64" --file rhel6.5-x86_64.raw --disk-format raw --container-format bare --is-public True --progress

3.flavor

h.bmsc
m.bmsc 

2	8192MB	30

nova flavor-create m.bmsc auto 4096 20 1 
nova flavor-create h.bmsc auto 8192 30 2

nova flavor-create m.bmc auto 2048 180 2
nova flavor-create h.bmc_reporting auto 2048 180 2

bmc_reporting

4.公共网络

nid： 7badd8e2-1fb2-4f47-879e-99839078f00d

5. openstack endpoint public url 修改









