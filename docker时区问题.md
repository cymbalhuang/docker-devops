把时区设置加入到Dockerfile中
* Ubuntu
　　RUN echo “Asia/shanghai” 》 /etc/timezone;
* CentOS
　　RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  
  无dockerfile则bash进入docker容器后cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime，重启docker
