1. 创建文件dynamic_mount_docker_volume
```
#!/bin/sh
set -e
CONTAINER=$1
HOSTPATH=$2
CONTPATH=$3

REALPATH=$(readlink --canonicalize $HOSTPATH)
FILESYS=$(df -P $REALPATH | tail -n 1 | awk '{print $6}')
DEVICE=$(df -P $REALPATH | tail -n 1 | awk '{print $1}')

while read DEV MOUNT JUNK
do [ $DEV = $DEVICE ] && [ $MOUNT = $FILESYS ] && break
done </proc/mounts
[ $DEV = $DEVICE ] && [ $MOUNT = $FILESYS ] # Sanity check!

while read A B C SUBROOT MOUNT JUNK
do [ $MOUNT = $FILESYS ] && break
done < /proc/self/mountinfo
[ $MOUNT = $FILESYS ] # Moar sanity check!

SUBPATH=$(echo $REALPATH | sed s,^$FILESYS,,)
DEVDEC=$(printf "%d %d" $(stat --format "0x%t 0x%T" $DEV))

docker-enter $CONTAINER  sh -c \
             "[ -b $DEV ] || mknod --mode 0600 $DEV b $DEVDEC"
docker-enter $CONTAINER sh -c "[ -d /tmpmnt ] || mkdir /tmpmnt"
docker-enter $CONTAINER mount $DEV /tmpmnt
docker-enter $CONTAINER sh -c "[ -d $CONTPATH ] || mkdir -p $CONTPATH"
docker-enter $CONTAINER mount -o bind /tmpmnt/$SUBROOT/$SUBPATH $CONTPATH
docker-enter $CONTAINER umount /tmpmnt
docker-enter $CONTAINER  rmdir /tmpmnt
```
$ chmod +x dynamic_mount_docker_volume

$ docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter

$ ./dynamic_mount_docker_volume 955138b6c3ed /tmp/test /src

参考
1.https://phpor.net/blog/post/4576

2.https://github.com/pushiqiang/utils/tree/master/docker


