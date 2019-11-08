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

##### prometheus.yml
```
rule_files:
- /etc/config/rules
- /etc/config/alerts
scrape_configs:
- job_name: prometheus
  static_configs:
  - targets:
    - localhost:9090
- bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  job_name: kubernetes-apiservers
  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - action: keep
    regex: default;kubernetes;https
    source_labels:
    - __meta_kubernetes_namespace
    - __meta_kubernetes_service_name
    - __meta_kubernetes_endpoint_port_name
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    insecure_skip_verify: true
- bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  job_name: kubernetes-nodes
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - replacement: kubernetes.default.svc:443
    target_label: __address__
  - regex: (.+)
    replacement: /api/v1/nodes/${1}/proxy/metrics
    source_labels:
    - __meta_kubernetes_node_name
    target_label: __metrics_path__
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    insecure_skip_verify: true
- bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  job_name: kubernetes-nodes-cadvisor
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - replacement: kubernetes.default.svc:443
    target_label: __address__
  - regex: (.+)
    replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
    source_labels:
    - __meta_kubernetes_node_name
    target_label: __metrics_path__
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    insecure_skip_verify: true
- job_name: kubernetes-service-endpoints
  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - action: keep
    regex: true
    source_labels:
    - __meta_kubernetes_service_annotation_prometheus_io_scrape
  - action: replace
    regex: (https?)
    source_labels:
    - __meta_kubernetes_service_annotation_prometheus_io_scheme
    target_label: __scheme__
  - action: replace
    regex: (.+)
    source_labels:
    - __meta_kubernetes_service_annotation_prometheus_io_path
    target_label: __metrics_path__
  - action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    source_labels:
    - __address__
    - __meta_kubernetes_service_annotation_prometheus_io_port
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_service_label_(.+)
  - action: replace
    source_labels:
    - __meta_kubernetes_namespace
    target_label: kubernetes_namespace
  - action: replace
    source_labels:
    - __meta_kubernetes_service_name
    target_label: kubernetes_name
- honor_labels: true
  job_name: prometheus-pushgateway
  kubernetes_sd_configs:
  - role: service
  relabel_configs:
  - action: keep
    regex: pushgateway
    source_labels:
    - __meta_kubernetes_service_annotation_prometheus_io_probe
- job_name: kubernetes-services
  kubernetes_sd_configs:
  - role: service
  metrics_path: /probe
  params:
    module:
    - http_2xx
  relabel_configs:
  - action: keep
    regex: true
    source_labels:
    - __meta_kubernetes_service_annotation_prometheus_io_probe
  - source_labels:
    - __address__
    target_label: __param_target
  - replacement: blackbox
    target_label: __address__
  - source_labels:
    - __param_target
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_service_label_(.+)
  - source_labels:
    - __meta_kubernetes_namespace
    target_label: kubernetes_namespace
  - source_labels:
    - __meta_kubernetes_service_name
    target_label: kubernetes_name
- job_name: kubernetes-pods
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - action: keep
    regex: true
    source_labels:
    - __meta_kubernetes_pod_annotation_prometheus_io_scrape
  - action: replace
    regex: (.+)
    source_labels:
    - __meta_kubernetes_pod_annotation_prometheus_io_path
    target_label: __metrics_path__
  - action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    source_labels:
    - __address__
    - __meta_kubernetes_pod_annotation_prometheus_io_port
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - action: replace
    source_labels:
    - __meta_kubernetes_namespace
    target_label: kubernetes_namespace
  - action: replace
    source_labels:
    - __meta_kubernetes_pod_name
    target_label: kubernetes_pod_name

alerting:
  alertmanagers:
  - kubernetes_sd_configs:
      - role: pod
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
    - source_labels: [__meta_kubernetes_namespace]
      regex: prometheus-pq977
      action: keep
    - source_labels: [__meta_kubernetes_pod_label_app]
      regex: prometheus
      action: keep
    - source_labels: [__meta_kubernetes_pod_label_component]
      regex: alertmanager
      action: keep
    - source_labels: [__meta_kubernetes_pod_container_port_number]
      regex:
      action: drop
```


##### rules 报警规则
```
groups:
- name: node memory high 
  rules:
  - alert: node memory high 
    expr: 100 - ((node_memory_MemAvailable * 100) / node_memory_MemTotal) >90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: node memory high 
      description: "当前已使用内存大于 90%. 使用内存: {{ $value }}"

- name: Pod_all_cpu_usage
  rules:
  - alert: Pod_all_cpu_usage
    expr: (sum by(name) (rate(container_cpu_usage_seconds_total{image!="",name!="cocky_thompson",name!="nostalgic_engelbart"}[5m])) * 100)>75
    for: 5m
    labels:
      severity: error
    annotations:
      summary: Pod_all_cpu_usage
      description: "容器:{{ $labels.pod }} CPU 资源利用率大于 75% , (当前值:{{ $value }})"
      
- name: Pod_memory_usage
  rules:
  - alert: Pod_memory_usage
    expr: sum (container_memory_working_set_bytes{image!="",namespace!='ingress-nginx'}) by (pod_name)  > 1024*1024*1024*8
    for: 10m
    labels:
      severity: error
    annotations:
      summary:  Pod_memory_usage
      description: "容器 {{ $labels.name }} Memory 资源利用率大于 8G , (当前值: {{ $value }})"
      
- name: Pod_all_network_receive_usage
  rules:
  - alert: Pod_all_network_receive_usage
    expr: sum by (name)(irate(container_network_receive_bytes_total{container_name="POD"}[5m])) > 1024*1024*100
    for: 10m
    labels:
      severity: error
    annotations:
      summary: Pod_all_network_receive_usage
      description: "容器 {{ $labels.name }} network_receive 资源利用率大于 100M , (当前值:{{ $value }})"
      
- name: K8SNodeNotReady
  rules:
  - alert: K8SNodeNotReady
    expr: kube_node_status_condition{condition="Ready",status="true"} == 0
    for: 20m
    labels:
      severity: error
    annotations:
      summary: "Node error"
      description: "{{ $labels.node }} Kubelet 20分钟未与API通信,或者节点处于NotReady状态"
      
- name: Ingress50XErr
  rules:
  - alert: Ingress50XErr
    expr: sum(rate(nginx_ingress_controller_requests{status =~"5[0-9][1-9]"}[2m])) by (ingress) / sum(rate(nginx_ingress_controller_requests[2m])) by (ingress) > 20
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Ingress in 50x error"
      description: "{{$labels.ingress}}:  Ingress in 50x error  {{ $value }}"
      
- name: DaemonsetUnavailable
  rules:
  - alert: DaemonsetUnavailable
    expr: kube_daemonset_status_number_unavailable > 0
    for: 5m
    labels:
      severity: error
    annotations:
      summary: Daemonset Unavailable
      description: "{{$labels.daemonset}}:  出现异常  {{ $value }}"

- name: PodRestart
  rules:
  - alert: PodRestart
    expr: changes(kube_pod_container_status_restarts_total{pod !~ "analyzer.*"}[10m]) > 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: Pod Restart
      description: "{{$labels.pod}}{{ $labels.container }} 发生重启"
- name: PodDown
  rules:
  - alert: PodDown
    expr: kube_pod_status_phase{phase="Unknown"} == 1 or kube_pod_status_phase{phase="Failed"} == 1
    for: 1m
    labels:
      severity: error
    annotations:
      summary: Pod Down
      description: "{{$labels.pod}}{{ $labels.container }} 停止"
```


##### ops.tmpl 告警模版
```
{{ define "ops.default.message" }}
<table border="5">
<tr><td>报警项</td>
<td>节点</td>
<td>报警阀值</td>
<td>开始时间</td>
</tr>
{{ range $i, $alert := .Alerts }}
<tr><td>{{ index $alert.Labels "alertname" }}</td>
<td>{{ index $alert.Labels "instance" }}</td>
<td>{{ index $alert.Annotations "value" }}</td>
<td>{{ $alert.StartsAt }}</td>
</tr>
{{ end }}
</table>
{{ end }}
```
