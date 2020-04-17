方案一：

https://github.com/docker/compose/releases
下载docker-compose-Linux-x86_64，改名docker-compose，上传到/usr/local/bin/，加执行权限 chmod +x /usr/local/bin/docker-compose

方案二（不推荐）：

1、首先检查linux有没有安装python-pip包，终端执行 pip -V

2、没有python-pip包就执行命令 yum -y install epel-release

3、执行成功之后，再次执行yum -y install python-pip

4、对安装好的pip进行升级 pip install --upgrade pip

5、pip install docker-compose

注：第四步更新pip由于网络有时不稳定的原因，使用pip/pip3下载会出现网络不可达的问题,建议是修改pip源，使用国内镜像服务

pip install packagename -i http://pypi.douban.com/simple --trusted-host pypi.douban.com

参考原文：https://blog.csdn.net/hopygreat/article/details/78344933
