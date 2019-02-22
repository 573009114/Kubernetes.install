### 配置yum源
```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
### 禁用edge版本
yum-config-manager --disable docker-ce-edge


### 查询可更新docker版本列表
```
yum list docker-ce.x86_64  --showduplicates |sort -r
```

### 更新yum索引
```
yum makecache fast
```

### 升级docker版本
```
yum update docker-ce*
```
