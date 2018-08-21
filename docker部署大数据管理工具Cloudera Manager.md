docker pull cloudera/quickstart:latest

docker run --privileged=true -t -i -d --name cloudera --hostname=quickstart.cloudera -p 8020:8020 -p 80:80 -p 8888:8888 -p 7180:7180 -p 21050:21050 -p 50070:50070 -p 50075:50075 -p 50010:50010 -p 50020:50020 -p 60000:60000 -p 60010:60010 -p 60020:60020 -p 60030:60030 -p 4242:4242 cloudera/quickstart /usr/bin/docker-quickstart

docker exec -ti cloudera /home/cloudera/cloudera-manager --express

登录http://host:7180

如本机端口通，其他机器telnet不通，可能是机器没关闭selinux，必须关闭selinux，重启机器，才能访问
