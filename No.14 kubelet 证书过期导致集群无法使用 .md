##### 检查证书时间
```
openssl x509 -in /etc/kubernetes/pki/kubelet-client-current.pem -noout -text|grep ' Not '
```
##### 重启kubelet 生成 /etc/kubernetes/pki/kubelet-client-2019-10-16-17-50-25.pem 证书文件
##### 建立软链 ln -s /etc/kubernetes/pki/kubelet-client-2019-10-16-17-50-25.pem /etc/kubernetes/pki/kubelet-client-current.pem
##### 重启kubelet

##### 添加配置
```
KUBELET_HOSTNAME="--hostname-override=shyt-k8s-master2"
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
KUBELET_CONFIG="--config=/etc/kubernetes/kubelet-config.yml"
KUBELET_ARGS="--cluster-dns=10.254.0.10 --bootstrap-kubeconfig=/etc/kubernetes/kubelet-bootstrap.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cert-dir=/etc/kubernetes/pki --feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true --rotate-certificates=true"
```
