```
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: mytlssecret
  namespace: test
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
    ingress.kubernetes.io/ssl-redirect: "False"  #是否强制跳转
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
