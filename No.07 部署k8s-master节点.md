
### 下面的配置文件中，很多都是有ip的。注意留心看
####  配置 kube-apiserver CA
```
cat >kube-apiserver-csr.json<<EOF
{
    "CN": "kube-apiserver",
    "hosts": [
      "127.0.0.1",
      "192.168.44.138",
      "192.168.44.139",
      "192.168.44.140",
      "10.254.0.1",
      "172.30.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
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
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF


# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
```

####  配置 kube-controller-manager CA
```
cat >kube-controller-manager-csr.json<<EOF
{
    "CN": "system:kube-controller-manager",
    "hosts": [
      "127.0.0.1",
      "192.168.44.138",
      "192.168.44.139",
      "192.168.44.140"
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
            "O": "system:kube-controller-manager",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-controller-manager CA
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

#### 配置 kube-scheduler ca
```
cat >kube-scheduler-csr.json<<EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
      "127.0.0.1",
      "192.168.44.138",
      "192.168.44.139",
      "192.168.44.140"
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
            "O": "system:kube-scheduler",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-scheduler ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
ls kube-scheduler*.pem
```
#### 配置 kube-proxy ca
```
cat >kube-proxy-csr.json<<EOF
{
    "CN": "system:kube-proxy",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "system:kube-proxy",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-proxy ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
ls kube-proxy*.pem
```
#### 配置 admin ca
```
cat >admin-csr.json<<EOF
{
    "CN": "admin",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "system:masters",
            "OU": "System"
        }
    ]
}
EOF

# 生成 admin ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes admin-csr.json | cfssljson -bare admin
ls admin*.pem
```
# 复制生成的ca
```
mkdir -pv /etc/kubernetes/pki
cp ca*.pem admin*.pem kube-proxy*.pem kube-scheduler*.pem kube-controller-manager*.pem kube-apiserver*.pem /etc/kubernetes/pki
cd /etc/kubernetes && tar cvzf pki.tgz pki/
scp /etc/kubernetes/pki.tgz 其他机器:~/
```

#### 接下来开始正式部署过程
```
wget https://dl.k8s.io/v1.11.0/kubernetes-server-linux-amd64.tar.gz
tar xf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
mkdir -pv /usr/local/kubernetes-v1.11.0/bin
cp kube-apiserver kube-controller-manager kube-scheduler kube-proxy kubelet kubectl /usr/local/kubernetes-v1.11.0/bin
ln -sv /usr/local/kubernetes-v1.11.0 /usr/local/kubernetes
cp /usr/local/kubernetes/bin/kubectl /usr/local/bin/kubectl
kubectl version
```
#### 生成kubeconfig
```
# 使用 TLS Bootstrapping 
export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
cat > /etc/kubernetes/token.csv <<EOF
${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF

# 创建 kubelet bootstrapping kubeconfig
cd /etc/kubernetes
export KUBE_APISERVER="https://192.168.44.138:6443"
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kubelet-bootstrap.conf
kubectl config set-credentials kubelet-bootstrap \
  --token=${BOOTSTRAP_TOKEN} \
  --kubeconfig=kubelet-bootstrap.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=kubelet-bootstrap.conf
kubectl config use-context default --kubeconfig=kubelet-bootstrap.conf

# 创建 kube-controller-manager kubeconfig

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-controller-manager.conf
kubectl config set-credentials kube-controller-manager \
  --client-certificate=/etc/kubernetes/pki/kube-controller-manager.pem \
  --client-key=/etc/kubernetes/pki/kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-controller-manager.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-controller-manager \
  --kubeconfig=kube-controller-manager.conf
kubectl config use-context default --kubeconfig=kube-controller-manager.conf

# 创建 kube-scheduler kubeconfig

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-scheduler.conf
kubectl config set-credentials kube-scheduler \
  --client-certificate=/etc/kubernetes/pki/kube-scheduler.pem \
  --client-key=/etc/kubernetes/pki/kube-scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-scheduler.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-scheduler \
  --kubeconfig=kube-scheduler.conf
kubectl config use-context default --kubeconfig=kube-scheduler.conf

# 创建 kube-proxy kubeconfig

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.conf
kubectl config set-credentials kube-proxy \
  --client-certificate=/etc/kubernetes/pki/kube-proxy.pem \
  --client-key=/etc/kubernetes/pki/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.conf
kubectl config use-context default --kubeconfig=kube-proxy.conf

# 创建 admin kubeconfig

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=admin.conf
kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/pki/admin.pem \
  --client-key=/etc/kubernetes/pki/admin-key.pem \
  --embed-certs=true \
  --kubeconfig=admin.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=admin \
  --kubeconfig=admin.conf
kubectl config use-context default --kubeconfig=admin.conf

# 把 kube-proxy.conf 复制到其他节点
scp kubelet-bootstrap.conf kube-proxy.conf qita:/etc/kubernetes
scp kubelet-bootstrap.conf kube-proxy.conf qita:/etc/kubernetes
cd $HOME
```
#### 配置master相关组件
```
# 复制 etcd ca
mkdir -pv /etc/kubernetes/pki/etcd
cd /etc/etcd/ssl
cp etcd-ca.pem etcd-key.pem etcd.pem /etc/kubernetes/pki/etcd

# 生成 service account key
openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
ls /etc/kubernetes/pki/sa.*
cd $HOME

# 启动文件
cat >/etc/systemd/system/kube-apiserver.service<<EOF
[Unit]
Description=Kubernetes API Service
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/apiserver
ExecStart=/usr/local/kubernetes/bin/kube-apiserver \\
	    \$KUBE_LOGTOSTDERR \\
	    \$KUBE_LOG_LEVEL \\
	    \$KUBE_ETCD_ARGS \\
	    \$KUBE_API_ADDRESS \\
	    \$KUBE_SERVICE_ADDRESSES \\
	    \$KUBE_ADMISSION_CONTROL \\
	    \$KUBE_APISERVER_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 该配置文件同时被 kube-apiserver, kube-controller-manager
# kube-scheduler, kubelet, kube-proxy 使用
cat >/etc/kubernetes/config<<EOF
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=2"
EOF

cat >/etc/kubernetes/apiserver<<EOF
KUBE_API_ADDRESS="--advertise-address=192.168.44.138"
KUBE_ETCD_ARGS="--etcd-servers=https://192.168.44.138:2379,https://192.168.44.139:2379,https://192.168.44.140:2379 --etcd-cafile=/etc/kubernetes/pki/etcd/etcd-ca.pem --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
KUBE_ADMISSION_CONTROL="--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota"
KUBE_APISERVER_ARGS="--allow-privileged=true --authorization-mode=Node,RBAC --enable-bootstrap-token-auth=true --token-auth-file=/etc/kubernetes/token.csv --service-node-port-range=30000-32767 --tls-cert-file=/etc/kubernetes/pki/kube-apiserver.pem --tls-private-key-file=/etc/kubernetes/pki/kube-apiserver-key.pem --client-ca-file=/etc/kubernetes/pki/ca.pem --service-account-key-file=/etc/kubernetes/pki/sa.pub --enable-swagger-ui=true --secure-port=6443 --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --anonymous-auth=false --kubelet-client-certificate=/etc/kubernetes/pki/admin.pem --kubelet-client-key=/etc/kubernetes/pki/admin-key.pem --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/export/kube/audit.log --event-ttl=1h --apiserver-count=2"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
systemctl status kube-apiserver

# 浏览器访问测试
https://192.168.44.138:6443/swaggerapi
```

#### 配置启动kube-controller-manager

```
cat >/etc/systemd/system/kube-controller-manager.service<<EOF
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/controller-manager
ExecStart=/usr/local/kubernetes/bin/kube-controller-manager \\
	    \$KUBE_LOGTOSTDERR \\
	    \$KUBE_LOG_LEVEL \\
	    \$KUBECONFIG \\
	    \$KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

cat >/etc/kubernetes/controller-manager<<EOF
KUBECONFIG="--kubeconfig=/etc/kubernetes/kube-controller-manager.conf"
KUBE_CONTROLLER_MANAGER_ARGS="--address=127.0.0.1 --cluster-cidr=172.30.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem --service-account-private-key-file=/etc/kubernetes/pki/sa.key --root-ca-file=/etc/kubernetes/pki/ca.pem --leader-elect=true --use-service-account-credentials=true --node-monitor-grace-period=10s --pod-eviction-timeout=10s --allocate-node-cidrs=true --controllers=*,bootstrapsigner,tokencleaner"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
systemctl status kube-controller-manager
```

#### 配置启动kube-scheduler

```
cat >/etc/systemd/system/kube-scheduler.service<<EOF
[Unit]
Description=Kubernetes Scheduler Plugin
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/scheduler
ExecStart=/usr/local/kubernetes/bin/kube-scheduler \\
            \$KUBE_LOGTOSTDERR \\
            \$KUBE_LOG_LEVEL \\
            \$KUBECONFIG \\
            \$KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

cat >/etc/kubernetes/scheduler<<EOF
KUBECONFIG="--kubeconfig=/etc/kubernetes/kube-scheduler.conf"
KUBE_SCHEDULER_ARGS="--leader-elect=true --address=127.0.0.1"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
systemctl status kube-scheduler
```

#### 配置kubectl使用
```
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get no
```

#### 查看组件状态
```
kubectl get componentstatuses
```

#### 配置kubelet使用bootstrap
```
# 将 bootstrap token 文件中的 kubelet-bootstrap 用户赋予 system:node-bootstrapper cluster 角色
kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```
