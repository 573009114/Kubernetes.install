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

##### 2、证书授权ip提前规划
```
如：
"11.11.11.111",
"11.11.11.112",
"11.11.11.113"
```



### 创建etcd证书

```
# 写入配置
cat >etcd-ca-csr.json<<EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ]
}
EOF

# 生成 etcd root ca
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca

cat >etcd-csr.json<<EOF
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "11.11.11.111",
      "11.11.11.112",
      "11.11.11.113"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "etcd",
            "OU": "Etcd Security"
        }
    ]
}
EOF

# 生成 etcd ca
cfssl gencert -ca=etcd-ca.pem -ca-key=etcd-ca-key.pem -config=ca-config.json \
-profile=kubernetes etcd-csr.json | cfssljson -bare etcd
mkdir -pv /etc/etcd/ssl
cp etcd*.pem /etc/etcd/ssl
ls /etc/etcd/ssl/etcd*.pem

# 复制到其他节点
cd /etc/etcd && tar cvzf etcd-ssl.tgz ssl/

```

### 安装etcd
```
yum install -y etcd
```


### 配置etcd文件

```
ETCD_NAME=k8s-1
ETCD_DATA_DIR="/data/etcd/k8s-1"
ETCD_LISTEN_PEER_URLS="https://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="https://0.0.0.0:2379,https://0.0.0.0:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.44.138:2380"
ETCD_INITIAL_CLUSTER="k8s-1=https://192.168.44.138:2380,k8s-2=https://192.168.44.139:2380,k8s-3=https://192.168.44.140:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.44.138:2379,https://192.168.44.138:4001"
[Security]
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_DISCOVERY=""

```
### 授权文件

```
chown etcd:etcd /etc/etcd/ssl -R
chown etcd:etcd /data/etcd/ -R
```

### 集群检查
```
etcdctl  --endpoints "https://127.0.0.1:2379" --ca-file=/etc/etcd/ssl/etcd-ca.pem   --cert-file=/etc/etcd/ssl/etcd.pem   --key-file=/etc/etcd/ssl/etcd-key.pem   cluster-health
```
