参考官网https://docs.gitlab.com/ee/ci/

1.部署gitlab runner,以debug模式启动，用于输出错误详细信息

docker run -d --privileged=true --name gitlab-runner-debug --restart always -v /usr/local/lekang/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest --debug run

2.向gitlab注册runner

  1)进入runner容器docker exec -it gitlab-runner-debug bash
  
  2)使用命令gitlab-runner register注册，参考官网https://docs.gitlab.com/runner/register/
  
  注册输入token可以到组下面，使用组创建账号登录获取，同时以管事员账号登录管理后台，编辑新建runner,把Run untagged jobs打勾。
  
  一次重启机器后重新注册runner出现
  
  ERROR: Registering runner... failed                 runner=o7vXM8Ly status=couldn't execute POST against http://10.86.2.122:8011/api/v4/runners: Post http://10.86.2.122:8011/api/v4/runners: dial tcp 10.86.2.122:8011: i/o timeout
  
  网络不通有两个可能
  
  1)防火墙没开，可以执行下面命令打开，docker容器默认内网网段为172.17.0.0/16
  
  firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="172.17.0.0/16" port protocol="tcp" port="8011" accept"
  
  或
  
  iptables -I INPUT -i docker0 -j ACCEPT
  
  2）重建gitlab-runner容器提示warning ipv4 forwarding is disabled. networking will not work，原来是没打开ipv4 forward
  
  解决办法：
  
  vi /etc/sysctl.conf
  
  添加文本：
  
  net.ipv4.ip_forward=1
  
  重启network服务：
  
  systemctl restart network
  
  重启docker服务：
  
  systemctl restart docker
  
  3）Runner executor docker镜像可以有多个选择，这里使用docker容器来跑，镜像选择alpine，官方推荐一个最小化linux镜像，大小只有4M+，但是里面什么都没有，由于容器要用CI来跑maven打包，maven依赖java,还有ssh的命令处理，经常艰辛搜索及大神指点后使用一个带maven的alpine镜像，打入openssh sshpass包，用于传输容器中打出来的java包，最终镜像的Dockerfile为
    
    FROM docker.io/maven:alpine
    RUN apk update && apk add openssh-client openssh sshpass
    
   使用命令"docker build -t alpine-maven-sshpass ."打出镜像。
   镜像在注册runner时填入，或者编辑runner配置文件/etc/gitlab-runner/config.toml
    
   4）配置修改
    参考官网https://docs.gitlab.com/runner/configuration/advanced-configuration.html#advanced-configuration
    

    [[runners]]
    name = "runner-debug"
    url = "http://172.16.10.122:8011/"
    token = "ab8866790af9744c9294612e52f802"
    executor = "docker"
    clone_url = "http://172.16.10.122:8011"
    [runners.docker]
      tls_verify = false
      #image = "docker.io/maven:alpine"
      image = "alpine-maven-sshpass"
      privileged = false
      disable_cache = false
      volumes = ["/cache"]
      shm_size = 0
      extra_hosts = ["nexus.XX.XX:172.16.10.121"]
    [runners.cache]

    
   其中clone_url为替换IP，避免使用域名导致无法解析, 而extra_hosts允许用户配置host，用于解析域名，gitlab想得还真周到，一开始没看这份配置浪费好长时间
    想方法处理docker容器域名解析问题，由于此容器由gitlab runner容器内部启动，貌似无解
3. 编写项目中CI文件
  注意docker方式每个阶段由不同容器跑，所以这里打包部署使用一个容器，即不能分build,deploy跑
  
  ```
  office-build:
  stage: build
  script:
    - mvn -U -Dmaven.test.skip=true -f pom.xml clean package
    - sshpass -p 123456 ssh -o StrictHostKeyChecking=no root@$HOST_NAME "mkdir -p $PROJECT_PATH/$PROJECT_NAME"
    - sshpass -p 123456 scp -o "StrictHostKeyChecking no" $BUILD_PATH/$PROJECT_NAME/target/$PROJECT_NAME.jar root@$HOST_NAME:$PROJECT_PATH/$PROJECT_NAME
    - sshpass -p 123456 ssh -o StrictHostKeyChecking=no root@$HOST_NAME "cd $PROJECT_PATH/$PROJECT_NAME; /usr/local/java/jdk/bin/java -Dspring.profiles.active=dev -jar $PROJECT_NAME.jar"
  only:
    - /^.*develop$/
  ```
  
   打包，使用sshpass进程执行脚步，scp copy编译好的文件到目标运行机器，再运行程序。
    done.
