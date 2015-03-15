## 禁用security grope 和firewall

计算节点执行：

openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf DEFAULT security_group_api neutron

openstack-config --set /etc/neutron/plugin.ini securitygroup enable_security_group false
openstack-config --set /etc/neutron/plugin.ini securitygroup firewall_driver neutron.agent.firewall.NoopFirewallDriver

重启nova-compute 和 Neutron-openvswich-agent


网络节点执行：

openstack-config --set /etc/neutron/plugin.ini securitygroup enable_security_group false
openstack-config --set /etc/neutron/plugin.ini securitygroup firewall_driver neutron.agent.firewall.NoopFirewallDriver

重启Neutron 相关服务



## 启用security grope 和firewall

计算节点执行：

openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf DEFAULT security_group_api neutron


openstack-config --set /etc/neutron/plugin.ini securitygroup enable_security_group true
openstack-config --set /etc/neutron/plugin.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

重启nova-compute 和 Neutron-openvswich-agent


网络节点执行：

openstack-config --set /etc/neutron/plugin.ini securitygroup enable_security_group true
openstack-config --set /etc/neutron/plugin.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

重启Neutron 相关服务


##解决ipv4 通ipv6 不通问题





	
	