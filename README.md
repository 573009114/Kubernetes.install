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


#### No0-No8 集群搭建部分
#### 从No9 开始，后面的部分属于集群组件和使用技巧部分。
