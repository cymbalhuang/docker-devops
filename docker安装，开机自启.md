yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install -y docker-ce

service docker start

systemctl enable docker
