#### 安装指定版本docker
```
安装需要的依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2
配置稳定仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#更新并安装 Docker-CE
yum makecache fast
安装指定版本的Docker
yum list docker-ce --showduplicates | sort -r

yum install -y --setopt=obsoletes=0 \
docker-ce-17.03.1.ce-1.el7.centos \
docker-ce-selinux-17.03.1.ce-1.el7.centos
```
