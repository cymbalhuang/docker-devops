参考官网
http://dubbo.apache.org/books/dubbo-admin-book/install/admin-console.html
注意点
1.按官网下载源码，修改配置文件vi webapps/ROOT/WEB-INF/dubbo.properties
  修改其中dubbo.registry.address=zookeeper://127.0.0.1:2181
  替换zk地址，为dubbo.registry.address=zookeeper://zookeeper:2181
  zookeeper地址由后面启动容器时传入zk地址
2.打出war包dubbo-admin-2.0.0.war
3.创建Dockerfile内容如下
  FROM tomcat:8
  ADD dubbo-admin-2.0.0.war /usr/local/tomcat/webapps/
  CMD ["catalina.sh", "run"]
4.构建镜像docker build -t dubbo-ops .
5.运行容器docker run -d --name dubbo-ops --link=myzookeeper:zookeeper -p 8012:8080 dubbo-ops，其中link作用是传入zk地址
6.进入容器docker exec -it dubbo-ops bash，修改tomcat中webapp，把dubbo-admin-2.0.0.war中内容解压入ROOT中，删除其余内容，这步操作应该可以在Dockerfile处理，
  后面再深入研究
7.退出容器，重启容器，查看日志docker logs -f dubbo-ops
8.正常时可以将容器打包出新的镜像docker commit 52b644e05aeb dubbo-ops-v1，其中52b644e05aeb为容器id.
