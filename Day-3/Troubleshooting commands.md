- When I'm working on Ingress with Load Balancer I got stuck with accessing the Prometheus, Grafana and Alertmanager dashboards with errors. How I overcome with those errors and how I troubleshooted those errors.
### Below are the few troubleshooting commands for quick reference

- ALB Target Group Health Checks
```bash
aws elbv2 describe-target-health --target-group-arn <TARGET_GROUP_ARN>
```

- Confirm with namespace
```bash
kubectl get namespaces
```

- Verify Prometheus and Alertmanager Pods Exist
```bash
kubectl get pods -n monitoring
```

- Check Service Configuration
```bash
kubectl get svc -n monitoring
```

- Verify Ingress Routing for Prometheus & Alertmanager, Check what paths are exposed inside the cluster
```bash
kubectl get endpoints -n monitoring
```

- Check if Prometheus and Alertmanager are accessible within the cluster
```bash
kubectl run -it --rm --restart=Never busybox --image=busybox -- sh
```
- Inside the pod, test connectivity:
```bash
wget -qO- http://monitoring-kube-prometheus-prometheus.monitoring:9090
wget -qO- http://monitoring-kube-prometheus-alertmanager.monitoring:9093
```

- If these return 200 OK, then the services are working inside the cluster.
- If not, there’s a misconfiguration in the Service definitions.
