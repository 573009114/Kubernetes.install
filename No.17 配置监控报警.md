#### alertmanager.yml
```
global:
  smtp_smarthost: 'mail.XX.com.cn:25'
  smtp_from: 'alert@XX.com.cn'
  smtp_auth_username: 'alert'
  smtp_auth_password: 'XXXXXXXX'
  smtp_require_tls: false

templates:
  - '/etc/config/ops.tmpl'

route:
  receiver: 'ops-receiver'
  group_wait: 10s 
  group_interval: 10s  
  repeat_interval: 1h 
  group_by: [alertname]
  routes:
  - receiver: ops-receiver
    group_wait: 10s
    match_re:
      severity: severity
receivers:
- name: 'ops-receiver'
  email_configs:
  - to: 'hao.wen@XX.com.cn'
    headers: { Subject: " {{ .CommonAnnotations.summary }}" }
  - to: 'xiao.mingliang@XX.com.cn'
    headers: { Subject: " {{ .CommonAnnotations.summary }}" }
  - to : 'hou.ziqiang@XX.com.cn'
    headers: { Subject: " {{ .CommonAnnotations.summary }}" }
```


