# kubernetes 前期步骤
### 1、 centos7 升级4.4内核
1、导入key
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org  
```

2、安装elrepo的yum源
```
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm  
```
3、安装内核
```
yum --enablerepo=elrepo-kernel install  kernel-lt-devel kernel-lt 
```
4、默认启动的顺序是从0开始，新内核是从头插入（目前位置在0，而4.4.4的是在1），所以需要选择0。
```
grub2-set-default 0  
```
5、然后reboot重启，使用新的内核，下面是重启后使用的内核版本:
```
uname -r  
```
### 2、使用overlay2存储驱动

修改/etc/docker/daemon.json

```
{
  "registry-mirrors": ["https://registry.docker-cn.com"],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "live-restore": true
}
```
