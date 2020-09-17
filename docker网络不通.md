注意：centos的防火墙firewalld在docker使用中有问题，建议关闭firewalld，使用iptables防火墙，并且注意iptables本身无持久化功能，而docker过程中使用iptables控制网络，重启iptables会导致容易无法访问外网，可使用iptables-save > /etc/sysconfig/iptables命令持久化网络。如果iptables重启丢失规则，可重启docker容器，则自动添加规则

网络不通，重建gitlab-runner容器提示warning ipv4 forwarding is disabled. networking will not work，原来是没打开ipv4 forward

解决办法：

vi /etc/sysctl.conf

添加文本：

net.ipv4.ip_forward=1

重启network服务：

systemctl restart network

重启docker服务：

systemctl restart docker
