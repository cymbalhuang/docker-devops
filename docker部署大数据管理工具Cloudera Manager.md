docker pull cloudera/quickstart:latest

docker run --privileged=true -t -i -d --name cloudera --hostname=quickstart.cloudera -p 8020:8020 -p 80:80 -p 8888:8888 -p 7180:7180 -p 21050:21050 -p 50070:50070 -p 50075:50075 -p 50010:50010 -p 50020:50020 -p 60010:60010  cloudera/quickstart /usr/bin/docker-quickstart

docker exec -ti cloudera /home/cloudera/cloudera-manager --express

登录http://host:7180
