# 1、部署钉钉告警插件
```
kubectl -nprometheus apply -f dingtalk.yaml
```

# 2、配置altermanager
```
  alertmanager.yml:
    global:
      resolve_timeout: 5m
      # slack_api_url: ''

    receivers:
      - name: webhook
        webhook_configs:
        - url: 'http://dingtalk.prometheus.svc.cluster.local.:8060/dingtalk/webhook1/send'
          send_resolved: true

    route:
      group_wait: 10s
      group_by: ['webhook']
      group_interval: 10s
      receiver: webhook
      repeat_interval: 3h
      routes:
      - receiver: webhook
        group_wait: 10s
        match:
          group: ops
```
