虚拟机配置静态IP
vmware->编辑->虚拟网络编辑器
虚拟机使用桥接模式，桥接到主机网卡

修改配置文件/etc/sysconfig/network-scripts/ifcfg-ens33 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
#UUID=d7b6043d-149b-40b1-ac77-375f509b25ba
DEVICE=ens33
ONBOOT=yes
IPADDR=172.16.10.125
NETMASK=255.255.255.0
GATEWAY=172.16.10.1
DNS1=211.136.192.6
DNS2=120.196.165.24
重启网络服务
service network restart
