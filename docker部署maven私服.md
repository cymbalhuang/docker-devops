# 参考官网
https://store.docker.com/community/images/sonatype/nexus3
# 注意点
1.创建外部数据目录/apps/nexus,配置目录权限chown -R root:root /apps/nexus

2.摘取镜像后使用
docker run --privileged=true -d -p 8013:8081 --name nexus3 -v /apps/nexus:/nexus-data -e INSTALL4J_ADD_VM_PARAMS="-Xms512m -Xmx512m -XX:MaxDirectMemorySize=512m" sonatype/nexus3 
注意传入参数--privileged=true，不然docker中用户无权限操作外部目录/apps/nexus，日志提示创建失败，导致启动失败

3.nginx使用域名访问时登录有问题，问题出现在nginx配置上，加上proxy_set_header Host $host:80;即可，具体原因未明
```
  server {
          listen 80;
          server_name domain.com;

          location / {
                  proxy_set_header Host $host:80;
                  proxy_pass http://172.16.1.1:8013;
          }

          error_page 500 502 503 504 /50x.html;
          location = /50x.html {
                root html;
          }
        }
```

```
 [ERROR] [ERROR] Some problems were encountered while processing the POMs:
 [FATAL] Non-resolvable parent POM for com.appscomm.lekang:lekang-sys-service:0.0.1-SNAPSHOT: Could not transfer artifact org.springframework.boot:spring-boot-starter-parent:pom:2.0.1.RELEASE from/to central (http://nexus.projectx.cn:8090/repository/maven-public/): Not authorized and 'parent.relativePath' points at wrong local POM @ line 13, column 10
  @ 
```
出现上面错误时，要在nexus管理后台管理，Security->Anonymous Access中开启匿名下载。
  
