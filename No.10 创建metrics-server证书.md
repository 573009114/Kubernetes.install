```
cat > metrics-server-csr.json <<EOF
{
  "CN": "aggregator",
  "hosts": [],
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
      "OU": "4Paradigm"
    }
  ]
}
EOF

```
```
cfssl gencert -ca=/etc/kubernetes/pki/ca.pem -ca-key=/etc/kubernetes/pki/ca-key.pem -config=ca-config.json -profile=kubernetes metrics-server-csr.json|cfssljson -bare metrics-server
```

在apiserver配置中加入
```
--requestheader-client-ca-file=/etc/kubernetes/pki/ca.pem --proxy-client-cert-file=/etc/kubernetes/pki/metrics-server.pem  --proxy-client-key-file=/etc/kubernetes/pki/metrics-server-key.pem --requestheader-allowed-names=kube-apiserver-proxy --requestheader-extra-headers-prefix=X-Remote-Extra --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User
```
