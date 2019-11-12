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



向etcd写入集群信息
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

#### 安装flannel
```
yum install -y flannel
```


#### flannel的配置
```
FLANNEL_ETCD_ENDPOINTS="https://192.168.44.138:2379,https://192.168.44.139:2379,https://192.168.44.140:2379"
FLANNEL_ETCD_PREFIX="/kube-centos/network"
FLANNEL_OPTIONS="-etcd-cafile=/etc/etcd/ssl/etcd-ca.pem -etcd-certfile=/etc/flanneld/ssl/flanneld.pem -etcd-keyfile=/etc/flanneld/ssl/flanneld-key.pem"

```


