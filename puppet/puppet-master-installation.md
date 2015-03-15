## Puppet Master 安装

### 安装前检查

1.检查防火墙是否开启，打开8140端口

2.Name resolution 配置，master 和agent 都要有主机名并且唯一，配置放入/etc/hosts 文件

3.时间保持同步 ，安装配置ntpd 服务。

	yum install -y ntpd


### 安装puppet master
	
先安装puppet repo 包

	sudo rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm

安装puppet master

	sudo yum install puppet-server
	systectl start puppetmaster
	systemctl enable puppetmaster

### 安装puppet agent

	 yum install puppet
	 systemctl start puppet
	 systemctl enable puppet	  

### puppet 部署







