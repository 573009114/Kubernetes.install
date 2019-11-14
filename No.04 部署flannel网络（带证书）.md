[证书配置参考 etcd证书部分](https://github.com/573009114/Kubernetes.install/blob/master/No.03%20%E5%BF%AB%E9%80%9F%E9%83%A8%E7%BD%B2etcd%E6%9C%8D%E5%8A%A1%EF%BC%88%E5%B8%A6%E8%AF%81%E4%B9%A6%EF%BC%89.md)
 



###### 向etcd写入集群信息   # flannel 0.7.1 不支持etcd_api v3
##### etcd_api v2 写法：
```

etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --ca-file=/etc/etcd/ssl/etcd-ca.pem \
  --cert-file=/etc/etcd/ssl/etcd.pem  \
  --key-file=/etc/etcd/ssl/etcd-key.pem  \
  set /kube-centos/network/config '{"Network":"'172.30.0.0/16'", "SubnetLen": 24, "Backend": {"Type": "host-gw"}}'
 ```
 ##### etcd_api v3 写法：
 ```
ETCDCTL_API=3 etcdctl  \
        --endpoints "https://127.0.0.1:2379" \
        --cacert=/etc/etcd/ssl/etcd-ca.pem   \
        --cert=/etc/etcd/ssl/etcd.pem   \
        --key=/etc/etcd/ssl/etcd-key.pem \
 put /testk8s/network/config '{ "Network": "172.30.0.0/16" }'
 ```
 
 
#### 得到如下反馈信息
```
{"Network":"172.30.0.0/16", "SubnetLen": 24, "Backend": {"Type": "host-gw"}}
```

# yum安装flannel 方法：
```
yum install -y flannel
``` 

#### flannel的配置
```
FLANNEL_ETCD_ENDPOINTS="https://192.168.44.138:2379,https://192.168.44.139:2379,https://192.168.44.140:2379"
FLANNEL_ETCD_PREFIX="/kube-centos/network"
FLANNEL_OPTIONS="-etcd-cafile=/etc/etcd/ssl/etcd-ca.pem -etcd-certfile=/etc/flanneld/ssl/flanneld.pem -etcd-keyfile=/etc/flanneld/ssl/flanneld-key.pem"

```



# 二进制部署flannel方法：

##### 一，下载 flanneld 二进制文件
```
wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz
mkdir -pv /opt/flannel/{bin,etc}
tar -zxvf flannel-v0.11.0-linux-amd64.tar.gz -C /opt/flannel/bin/

```
##### 二，创建docker关联flannel子网关系
```
cat > /usr/lib/systemd/system/docker.service.d/flannel.conf << "EOF"
[Service]
EnvironmentFile=-/run/flannel/docker
EOF 
```

##### 三，配置flannel
```
cat /opt/flannel/etc/flannel.conf

FLANNEL_ETCD="-etcd-endpoints=https://10.20.55.171:2379,https://10.20.55.172:2379,https://10.20.55.173:2379"
FLANNEL_ETCD_KEY="-etcd-prefix=/kube-centos/network"
FLANNEL_ETCD_CAFILE="--etcd-cafile=/etc/etcd/ssl/etcd-ca.pem"
FLANNEL_ETCD_CERTFILE="--etcd-certfile=/etc/etcd/ssl/etcd.pem"
FLANNEL_ETCD_KEYFILE="--etcd-keyfile=/etc/etcd/ssl/etcd-key.pem"

```

##### 四，配置systemd 启动
```
cat  /usr/lib/systemd/system/flannel.service

[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/opt/flannel/etc/flannel.conf
EnvironmentFile=-/etc/sysconfig/docker-network
ExecStart=/opt/flannel/bin/flanneld ${FLANNEL_ETCD} ${FLANNEL_ETCD_KEY} ${FLANNEL_ETCD_CAFILE} ${FLANNEL_ETCD_CERTFILE} ${FLANNEL_ETCD_KEYFILE}
ExecStartPost=/opt/flannel/bin/mk-docker-opts.sh -k $DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure

[Install]
WantedBy=multi-user.target
WantedBy=docker.service


启动
systemctl daemon-reload && systemctl enable flannel && systemctl restart flannel
```


