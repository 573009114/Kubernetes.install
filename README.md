#### 本文档全是本人在实际搭建过程中和工作中遇到的问题记录，欢迎大家补充或者是提出疑问 
##### qq: 573009114
##### 微信：qixian_wenshao

### 常见错误
1、 创建coredns时pod持续创建中，无法完成。（排除网络问题）
解决：
```
kubectl create clusterrolebinding kubelet-node-clusterbinding --clusterrole=system:node --group=system:nodes
```

```
kubectl describe clusterrolebindings kubelet-node-clusterbinding
Name:         kubelet-node-clusterbinding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  system:node
Subjects:
  Kind   Name          Namespace
  ----   ----          ---------
  Group  system:nodes
  
  ```


2、 如果在1.16.x版本中，发现apiserver日志出现以下错误
```
Nov 15 10:03:39 yhg-k8s-55171 kube-apiserver: I1115 10:03:39.889451    9303 node_authorizer.go:253] NODE DENY: yhg-k8s-55174 &authorizer.AttributesRecord{User:(*user.DefaultInfo)(0xc009c24e00), Verb:"update", Namespace:"kube-node-lease", APIGroup:"coordination.k8s.io", APIVersion:"v1", Resource:"leases", Subresource:"", Name:"yhg-k8s-55175", ResourceRequest:true, Path:"/apis/coordination.k8s.io/v1/namespaces/kube-node-lease/leases/yhg-k8s-55175"}
```
则需要绑定权限和组
```
kubectl create clusterrolebinding bootstrap --clusterrole=cluster-admin --user=kubelet-bootstrap
```
修改kubelet参数，将kubelet-bootstrap改成下面的这种
```
--bootstrap-kubeconfig=/etc/kubernetes/bootstrap.conf
```


#### No0-No8 集群搭建部分
#### 从No9 开始，后面的部分属于集群组件和使用技巧部分。
