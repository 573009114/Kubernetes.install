### 第一种方法： 根据文件创建secret ，默认会创建到default 命名空间
```
kubectl create secret tls https-secret --key tls.key --cert tls.crt

```
### 第二种方法： 使用yaml创建secret， 需要指定namespace（建议使用该种方式）
```
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: mytlssecret
  namespace: kube-system
data:
  tls.crt: <base64 encoded cert>
  tls.key: <base64 encoded key>
```
##### crt 和key 需要使用base64 转化一下
##### 每个namespace 都需要一个secret


### 下面是ingress配置的例子：
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    ingress.kubernetes.io/ssl-redirect: "true"  #是否强制跳转
spec:
  tls:
  - hosts:
    - dashboard-docker.xxx.com.cn
    secretName: mytlssecret
  rules:
  - host: dashboard-docker.xxx.com.cn
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: dashboard

```
