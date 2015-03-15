## Foreman Installation

### 安装foreman

先安装foreman 源

	yum -y install http://yum.theforeman.org/releases/1.1/el6/x86_64/foreman-release.rpm

	yum -y install foreman-installer 

如果手动配置了本地源,开 foreman-configure-scl-repo 参数为false，否则默认会用重新安装epel 等相关yum 源。

	foreman-installer --foreman-configure-scl-repo false

安装完成且成功显示类似如下输出

	[root@config ~]# foreman-installer --foreman-configure-scl-repo false
	Installing             Done                                               [100%] [.................................................................................]
	  Success!
	  * Foreman is running at https://config.cnsheta.com
	      Initial credentials are admin / qf7BVhc2vvbjDd7N
	  * Foreman Proxy is running at https://config.cnsheta.com:8443
	  * Puppetmaster is running at port 8140
	  The full log is at /var/log/foreman-installer/foreman-installer.log
	[root@config ~]# foreman-installer --foreman-configure-scl-repo false
	Installing             Done                                               [100%] [.................................................................................]
	  Success!
	  * Foreman is running at https://config.cnsheta.com
	      Initial credentials are admin / qf7BVhc2vvbjDd7N
	  * Foreman Proxy is running at https://config.cnsheta.com:8443
	  * Puppetmaster is running at port 8140
	  The full log is at /var/log/foreman-installer/foreman-installer.log


默认监听80和443端口，打开浏览器http://config.cnsheta.com/ 或者  https://config.cnsheta.com ,用户名：admin,密码：qf7BVhc2vvbjDd7N ，登录后修改密码。

### foreman 初始化配置

添加一个smart prox,

	登录dashboard 后，依次进入 infrastructure -> smart proxies -> new smart proxies ,添加smart proxy 的url 地址为 https://config.cnsheta.com:8443

### 配置foreman 支持 OS Provision support。 

登录dashboard 后，依次进入 infrastructure -> provision setup -> new provision setup , 按导航依次配置os provision 环境。

需要特别注意的是：network config 部分 ，Gateway address  和 Primary DNS server 都必须要配置，目前有bug 如果不填生成的dhcpd.conf 配置文件有语法错误。

最后生成foreman-installer 新的配置参数命令行，重新配置foreman ，比如这样：

	foreman-installer \
	  --enable-foreman-proxy \
	  --foreman-proxy-tftp=true \
	  --foreman-proxy-tftp-servername=192.168.1.111 \
	  --foreman-proxy-dhcp=true \
	  --foreman-proxy-dhcp-interface=eno1 \
	  --foreman-proxy-dhcp-gateway=192.168.1.1 \
	  --foreman-proxy-dhcp-range="192.168.1.2 192.168.1.254" \
	  --foreman-proxy-dhcp-nameservers="192.168.1.111" \
	  --foreman-proxy-dns=true \
	  --foreman-proxy-dns-interface=eno1 \
	  --foreman-proxy-dns-zone=cnsheta.com \
	  --foreman-proxy-dns-reverse=1.168.192.in-addr.arpa \
	  --foreman-proxy-foreman-base-url=https://config.cnsheta.com \
	  --foreman-proxy-oauth-consumer-key=xrDcifd5f7ZCY3wBNXGFuFJE6fyMjLRL \
	  --foreman-proxy-oauth-consumer-secret=DkFe96446bFesbP2EVvwEeyPrw3LuhiH \
	  --foreman-configure-scl-repo false



### 添加 puppetlabs-puppet module

	puppet module install puppetlabs-puppet
