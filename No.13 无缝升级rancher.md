### 第一步：新创建rancher
```
docker run -d --restart=unless-stopped -p 8080:8080  -p 3443:443 --dns 10.x.56.9 rancher/server
```
### 第二步：创建容器卷
```
docker create --volumes-from 8902990e9144 --name rancher-data rancher/server
```
### 第三步：启动容器
```
docker run -d --volumes-from rancher-data --restart=unless-stopped -p 8089:80 -p 3443:443 --dns 10.52.56.9 rancher/server
```
