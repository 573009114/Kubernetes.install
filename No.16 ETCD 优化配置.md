```
[Member]
ETCD_NAME=pg-docker-55161
ETCD_DATA_DIR="/export/etcd/pg-docker-55161"
ETCD_LISTEN_PEER_URLS="https://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="https://0.0.0.0:2379,https://0.0.0.0:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.20.55.161:2380"
ETCD_INITIAL_CLUSTER="pg-docker-55161=https://10.20.55.161:2380,pg-docker-55162=https://10.20.55.162:2380,pg-docker-55163=https://10.20.55.163:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://10.20.55.161:2379,https://10.20.55.161:4001"
[Security]
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_DISCOVERY=""

ETCD_QUOTA_BACKEND_BYTES="8589934592"
# 解决 内存2g上限的问题
ETCD_MAX_TXN_OPS="33554432"
# 解决服务器将接受的最大客户端请求大小为1.5M的问题
ETCD_AUTO_COMPACTION_RETENTION="1"
# 开启自动压缩

### 参考 https://www.cnblogs.com/linuxws/p/11194403.html
```


### k8s 自带etcd的配置方式，修改/etc/kubernetes/manifests/etcd.yaml
```
- etcd
- --advertise-client-urls=https://10.10.7.249:2379
- --cert-file=/etc/kubernetes/pki/etcd/server.crt
- --client-cert-auth=true
- --data-dir=/data/etcd
- --initial-advertise-peer-urls=https://10.10.7.249:2380
- --initial-cluster=vm10-10-7-99=https://10.10.7.99:2380,vm10-10-7-249=https://10.10.7.249:2380
- --initial-cluster-state=existing
- --key-file=/etc/kubernetes/pki/etcd/server.key
- --listen-client-urls=https://127.0.0.1:2379,https://10.10.7.249:2379
- --listen-metrics-urls=http://127.0.0.1:2381
- --listen-peer-urls=https://10.10.7.249:2380
- --name=vm10-10-7-249
- --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
- --peer-client-cert-auth=true
- --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
- --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
- --snapshot-count=10000
- --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
- --quota-backend-bytes=8589934592
- --max-request-bytes=10485760
- --auto-compaction-retention=1
```
