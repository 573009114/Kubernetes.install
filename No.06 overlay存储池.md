### 概述：

一般来说，overlay/overlay2驱动能提供更好的性能。可以肯定的是，比aufs和devicemapper性能更优，有些特定环境下，其性能甚至比btrfs更好。

iNode限制。使用overlay存储驱动会导致iNode的过渡消耗，尤其是宿主机上的容器和镜像数量快速增长时。保存大量容器的宿主机和过多的启停容器会耗尽iNodes。而overlay2则不存在这个问题。

不幸的是，我们仅仅创建文件系统时指定inodes大小，正因为如此，建议将/var/lib/docker目录放在独立设备上，使其拥有自己的文件系统，或者在创建文件系统时手工指定文件系统大小。

### 参考https://github.com/NVIDIA/k8s-device-plugin#preparing-your-gpu-nodes 支持GPU


### 使用overlay2存储驱动

修改/etc/docker/daemon.json

```
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "dns":[
      "10.52.0.31",
      "10.52.0.32"
  ],

  "live-restore": true
}
```


### 修改docker启动脚本

vi /usr/lib/systemd/system/docker.service
```
ExecStart=/usr/bin/dockerd  $DOCKER_NETWORK_OPTIONS
```
