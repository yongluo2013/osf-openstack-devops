##openstack dvr 配置

网络节点 节点配置

openstack-config --set /etc/neutron/neutron.conf DEFAULT dvr_base_mac  fa:16:3f:00:00:00
openstack-config --set /etc/neutron/neutron.conf DEFAULT router_distributed  True


openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch,l2population
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent l2_population True
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent arp_responder  True
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent enable_distributed_routing  True

openstack-config --set /etc/neutron/l3_agent.ini DEFAULT agent_mode dvr_snat


计算节点配置


openstack-config --set /etc/neutron/neutron.conf DEFAULT dvr_base_mac  fa:16:3f:00:00:00
openstack-config --set /etc/neutron/neutron.conf DEFAULT router_distributed  True

openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch,l2population

openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent l2_population True
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent arp_responder  True
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent enable_distributed_routing  True


openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent enable_distributed_routing True
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent arp_responder  True

openstack-config --set /etc/neutron/l3_agent.ini DEFAULT agent_mode dvr
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.OVSInterfaceDriver
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT use_namespaces  True
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT external_network_bridge br-ex
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT verbose True
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT debug True

ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth2
ifconfig br-ex up
ifconfig br-tun up
ifconfig br-int up

参考“

http://docs.openstack.org/networking-guide/content/ha-dvr.html
https://ask.openstack.org/en/question/50969/configuring-neutron-in-juno-for-ha/

精华帖子：
https://wiki.openstack.org/wiki/Neutron/DVR_L2_Agent
http://blog.csdn.net/matt_mao/article/details/39180135
http://m.blog.csdn.net/blog/canxinghen/40978983

https://docs.google.com/presentation/d/1ktCLAdglpKdsC--fQ2F_c2d3X1u-lyRGGUzRu9ITuWo/edit#slide=id.g2b6a14e30_036
https://docs.google.com/presentation/d/1ktCLAdglpKdsC--fQ2F_c2d3X1u-lyRGGUzRu9ITuWo/edit#slide=id.g2b6a14e30_030

详解dvr 和 l3 HA

https://kimizhang.wordpress.com/2014/11/25/building-redundant-and-distributed-l3-network-in-juno/


openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent l2_population False
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent arp_responder  False
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent arp_responder  False
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent l2_population False


openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch,l2population
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent l2_population True
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent arp_responder  True

openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent arp_responder  True
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent l2_population True

openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini ovs enable_tunneling True
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini ovs integration_bridge  br-int
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini ovs tunnel_bridge  br-tun


openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini ovs local_ip 10.0.1.20



openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent l2_population False
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini agent arp_responder  False

openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent arp_responder  False
openstack-config --set /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini agent l2_population False



问题：
dashboard 不能 添加floating ip
https://ask.openstack.org/en/question/51634/juno-dvr-associate-floating-ip-reported-no-ports-available/
