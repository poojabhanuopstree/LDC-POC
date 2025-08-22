apiVersion: operator.victoriametrics.com/v1beta1
kind: VMRule
metadata:
  name: loki-health-rules
  namespace: monitoring # Change to your VictoriaMetrics namespace
spec:
  groups:
  - name: loki-component-health
    rules:
    - alert: LokiQueryLatencyHigh
      expr: |
        histogram_quantile(0.99, sum(rate(loki_request_duration_seconds_bucket[5m])) by (le)) > 10
      for: 5m
      labels:
        severity: warning
        channel: slack
        team: devops
        alert: loki
      annotations:
        summary: "Loki query latency is high"
        description: "The 99th percentile of Loki query duration has exceeded 10 seconds. Users are experiencing slow query performance."

    - alert: LokiCacheCorruptionDetected
      expr: |
        increase(loki_cache_corrupt_chunks_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
        channel: slack
        team: devops
        alert: loki
      annotations:
        summary: "Loki has detected corrupt chunks in the cache"
        description: "Loki has found one or more corrupt chunks in the cache. This could indicate a serious problem with data integrity."

    - alert: LokiCacheQueueFull
      expr: |
        loki_cache_background_queue_length > 100
      for: 2m
      labels:
        severity: warning
        channel: slack
        team: devops
        alert: loki
      annotations:
        summary: "Loki cache background queue is full"
        description: "The cache background queue is backing up, which could lead to performance degradation or dropped writes."

    - alert: LokiServicePanicked
      expr: |
        increase(loki_panic_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
        channel: slack
        team: devops
        alert: loki
      annotations:
        summary: "Loki service panic detected"
        description: "A Loki component has panicked. Immediate investigation is required."

    - alert: LokiRequestOverload
      expr: |
        loki_inflight_requests > 100
      for: 5m
      labels:
        severity: warning
        channel: slack
        team: devops
        alert: loki
      annotations:
        summary: "Loki is experiencing high request load"
        description: "The number of inflight requests is high, suggesting Loki may be overloaded."
