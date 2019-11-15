# 配置node相关组件

## 安装cni
```
# 安装 cni
cd /server/software/k8s
wget https://github.com/containernetworking/plugins/releases/download/v0.7.1/cni-plugins-amd64-v0.7.1.tgz
mkdir -pv /opt/cni/bin
tar xf cni-plugins-amd64-v0.7.1.tgz -C /opt/cni/bin
ls -l /opt/cni/bin
cd $HOME

```
## 配置 cni支持
```
创建/etc/cni/net.d/10-flannel.conflist
{
  "cniVersion": "0.3.0",
  "name": "cbr0",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

## 配置启动kubelet
```
# 启动文件
mkdir -pv /data/kubelet
cat >/etc/systemd/system/kubelet.service<<EOF
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/data/kubelet
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kubelet
ExecStart=/usr/local/kubernetes/bin/kubelet \\
            \$KUBE_LOGTOSTDERR \\
            \$KUBE_LOG_LEVEL \\
            \$KUBELET_CONFIG \\
            \$KUBELET_HOSTNAME \\
            \$KUBELET_POD_INFRA_CONTAINER \\
            \$KUBELET_ARGS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

cat >/etc/kubernetes/config<<EOF
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=2"
EOF

# 注意修改相关ip
cat >/etc/kubernetes/kubelet<<EOF
KUBELET_HOSTNAME="--hostname-override=192.168.44.138"
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
KUBELET_CONFIG="--config=/etc/kubernetes/kubelet-config.yml"
KUBELET_ARGS="--cluster-dns=10.20.53.10,8.8.8.8 --bootstrap-kubeconfig=/etc/kubernetes/kubelet-bootstrap.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cert-dir=/etc/kubernetes/pki --network-plugin=cni --cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d"
EOF

# 注意修改相关ip
# lab1 lab2 lab3 使用各自ip
cat >/etc/kubernetes/kubelet-config.yml<<EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 192.168.44.138
port: 10250
cgroupDriver: cgroupfs
clusterDNS:
  - 10.254.0.10
clusterDomain: cluster.local.
hairpinMode: promiscuous-bridge
serializeImagePulls: false
authentication:
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
EOF

# 启动
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet
通过证书请求
# 在配置了kubectl的节点上执行如下操作

# 查看
kubectl get csr

# 通过
kubectl certificate approve node-csr-Yiiv675wUCvQl3HH11jDr0cC9p3kbrXWrxvG3EjWGoE

# 查看节点
# 此时节点状态为 NotReady
kubectl get nodes

# 在node节点查看生成的文件
ls -l /etc/kubernetes/kubelet.conf
ls -l /etc/kubernetes/pki/kubelet*
```


## 配置启动kube-proxy


#### 安装
```
yum install -y conntrack-tools
```
```
# 启动文件
cat >/etc/systemd/system/kube-proxy.service<<EOF
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/proxy
ExecStart=/usr/local/kubernetes/bin/kube-proxy \\
	    \$KUBE_LOGTOSTDERR \\
	    \$KUBE_LOG_LEVEL \\
	    \$KUBECONFIG \\
	    \$KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 注意修改相关ip
# lab1 lab2 lab3 使用各自ip
# 由于 1.11.0 ipvs 在centos7上有bug无法正常使用
# 实验使用 iptables 模式
# 以后版本可以使用 ipvs 模式
cat >/etc/kubernetes/proxy<<EOF
KUBECONFIG="--kubeconfig=/etc/kubernetes/kube-proxy.conf"
KUBE_PROXY_ARGS="--bind-address=192.168.44.138 --proxy-mode=iptables --hostname-override=192.168.44.138 --cluster-cidr=172.30.0.0/16"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
systemctl status kube-proxy
```

## 设置集群角色
##### 设置 lab1 为 master
```
kubectl label nodes 192.168.44.138 node-role.kubernetes.io/master=
```
#### 设置 lab2 lab3 为 node
```
kubectl label nodes 192.168.44.139 node-role.kubernetes.io/node=
kubectl label nodes 192.168.44.140 node-role.kubernetes.io/node=
```
#### 设置 master 一般情况下不接受负载
```
kubectl taint nodes 192.168.44.138 node-role.kubernetes.io/master=true:NoSchedule
```

## 查看节点
#### 此时节点状态为 NotReady
```
kubectl get no
```
