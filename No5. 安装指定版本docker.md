#### 安装指定版本docker
```
安装需要的依赖包
yum install -y yum-utils device-mapper-persistent-data
配置稳定仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
安装指定版本的Docker
yum list docker-ce --showduplicates | sort -r

yum install -y --setopt=obsoletes=0 \
docker-ce-17.03.1.ce-1.el7.centos \
docker-ce-selinux-17.03.1.ce-1.el7.centos
```
