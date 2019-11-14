### 基础条件
##### 1、下载cffsl等命令
```
# 下载
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64

# 安装
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl*

```




#### flannel 专用证书
```
cat > flanneld-csr.json <<EOF
{
  "CN": "flanneld",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "4Paradigm"
    }
  ]
}
EOF
```

生成证书
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes flanneld-csr.json |cfssljson -bare flanneld

mkdir /etc/flanneld/ssl -pv
mv flanneld*.pem /etc/flanneld/ssl/
```



向etcd写入集群信息   # flannel 0.7.1 不支持etcd_api v3
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
##### 二，创建remove-docker0.sh
```
cat > /opt/flannel/bin/remove-docker0.sh << "EOF"
#!/bin/bash
# Delete default docker bridge, so that docker can start with flannel network.
# exit on any erro
set -e
 
rc=0
ip link show docker0 > /dev/null 2>&1 || rc="$?"
if [[ "$rc" -eq "0" ]];then
ip link set dev docker0 down
ip link delete docker0
fi
EOF

chmod +x /opt/flannel/bin/remove-docker0.sh 
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
Before=docker.service
[Service]
EnvironmentFile=-/opt/flannel/etc/flannel.conf
ExecStartPre=/opt/flannel/bin/remove-docker0.sh
ExecStart=/opt/flannel/bin/flanneld ${FLANNEL_ETCD} ${FLANNEL_ETCD_KEY} ${FLANNEL_ETCD_CAFILE} ${FLANNEL_ETCD_CERTFILE} ${FLANNEL_ETCD_KEYFILE}
ExecStartPost=/opt/flannel/bin/mk-docker-opts.sh -d /run/flannel/docker
Type=notify
[Install]
WantedBy=multi-user.target
RequiredBy=docker.service

启动
systemctl daemon-reload && systemctl enable flannel && systemctl restart flannel
```


