##### 检查证书时间
```
openssl x509 -in /etc/kubernetes/pki/kubelet-client-current.pem -noout -text|grep ' Not '
显示：
            Not Before: Oct 16 09:43:00 2018 GMT
            Not After : Oct 15 09:43:00 2019 GMT

```
##### 重启kubelet 
```
通过master 节点 kubectl get csr 查看到有
NAME                                                   AGE       REQUESTOR           CONDITION
node-csr-8TOYc_WLFsHVVMnugCF8oQjNxInYrTvujZlqW-kmbrg   20h       kubelet-bootstrap   Pending
node-csr-OcvdtL2ps2DXw8Mu0ioBEx7Rw2U1opsWjQJgawHZt94   20h       kubelet-bootstrap   Pending

然后 
kubectl certificate approve node-csr-8TOYc_WLFsHVVMnugCF8oQjNxInYrTvujZlqW-kmbrg 
之后会生成
/etc/kubernetes/pki/kubelet-client-2019-10-16-17-50-25.pem 证书文件
```
##### 建立软链 
```
ln -s /etc/kubernetes/pki/kubelet-client-2019-10-16-17-50-25.pem /etc/kubernetes/pki/kubelet-client-current.pem
```
##### 重启kubelet

##### 添加kubelet配置
```
KUBELET_ARGS="--feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true --rotate-certificates=true"
```
##### 添加controller-manager配置
```
KUBE_CONTROLLER_MANAGER_ARGS="--feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true --experimental-cluster-signing-duration=87600h0m0s"

```
