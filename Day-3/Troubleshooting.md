*While working on Ingress with a Load Balancer, I encountered issues accessing the Prometheus, Grafana, and Alert manager dashboards due to various errors. I troubleshot these issues systematically and successfully resolved them. Below is a detailed explanation of the errors I faced and the steps I took to overcome them.*

## ERROR 1
- AccessDenied

## AccessDenied Error: Create an IAM policy with below configuration.

**A.** Check the node group IAM role which is attached to the EC2 instance.

**B.** Go to policy and create, and name it as "AWSLoadBalancerControllerIAMPolicy"
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInstances",
                "ec2:DescribeVpcs",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:CreateRule",
                "elasticloadbalancing:DeleteRule"
            ],
            "Resource": "*"
        }
    ]
}
```
**C.** Attach the created policy to the role and run below commands:
```bash
kubectl delete pod -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```
or
```bash
kubectl rollout restart deployment aws-load-balancer-controller -n kube-system
```
**Note:** The difference between the above two commands
```bash
kubectl delete pod -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```
- This forces the deletion of all pods matching the label (app.kubernetes.io/name=aws-load-balancer-controller).
- The deployment automatically creates new pods to replace the deleted ones.
- Useful when you want to immediately restart the ALB Controller.

✅ Best for: Forcing an immediate pod restart when you’ve made permission or role changes.

```bash
kubectl rollout restart deployment aws-load-balancer-controller -n kube-system
```
- This gracefully restarts the deployment by creating new pods before terminating old ones.
- Ensures a smooth transition without downtime.
- Relies on Kubernetes to handle rolling updates.

✅ Best for: Applying configuration changes without downtime (e.g., updating environment variables, annotations, or roles).

Which one should you use?
- If you just updated IAM permissions (like attaching a new IAM policy) → Use kubectl delete pod
- If you made configuration changes (like modifying Helm values) → Use kubectl rollout restart deployment

```bash 
kubectl get ingress -n monitoring
```

D. If the above commands doesn't work then try below one, make sure that you copied the arn of the policy and role name:
```bash
aws iam attach-role-policy --role-name <copy the role name> --policy-arn <copy the policy arn>
```
- Then following step C, then you should see the ADDRESS of your ALB IP.

## ERROR 2
- The ALB URL's didn't work for prometheus and grafana - it say's site can't be reached and for alertmanager it say's 502 bad gate way

1️⃣ Verify Service Status
Ensure that all your services (Prometheus, Grafana, and Alertmanager) are running correctly and have healthy endpoints.
```bash
kubectl get pods -n monitoring
```
Ensure that the pods for Prometheus, Grafana, and Alertmanager are running and healthy.

2️⃣ Check Service Definitions
Verify that the services for Prometheus, Grafana, and Alertmanager exist and are properly set up with the correct ports (e.g., Prometheus on port 9090, Grafana on port 80, Alertmanager on port 9093).
```bash
kubectl get svc -n monitoring
```
3️⃣ Check ALB Target Group Health
- Verify the health of the ALB target groups in the AWS Management Console.

4️⃣ Check for Ingress Controller Logs
```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```
## ERROR 3
- Later I changed the annotations in the monitoring-ingress file
```bash
    alb.ingress.kubernetes.io/healthcheck-path: /prometheus/-/healthy  # Prometheus health check
    alb.ingress.kubernetes.io/healthcheck-path: /alertmanager/-/healthy  # Alertmanager health check
    alb.ingress.kubernetes.io/healthcheck-path: /api/health  # Grafana health check
```
- With the above changes I can access to the Grafana but I cannot access Prometheus (404 page not found) and Alertmanager (502 bad Gateway)

## ERROR 4
- Later I got an issue with the alertmanager, like it is crashing everytimg
## Step 1: Edit the StatefulSet
```bash
kubectl get pods -n monitoring
```
- Updated the Alertmanager Arguments
```bash
kubectl edit statefulset alertmanager-monitoring-kube-prometheus-alertmanager -n monitoring
```
- Find this line:
```bash
--web.external-url=/alertmanager
```
- Modify it to:
```bash
--web.external-url=http://alertmanager.monitoring.svc:9093
```
- In case if you're using an external domain:
```bash
--web.external-url=https://your-alertmanager-domain
```
Save and exit the editor.
## Step 2: Restart Alertmanager
```bash
kubectl delete pod -n monitoring -l app.kubernetes.io/name=alertmanager
```
## Step 3: Verify Logs
```bash
kubectl get pods -n monitoring
kubectl logs -n monitoring -l app.kubernetes.io/name=alertmanager
```
# SOLUTION
## Step 1: Verify Application Configuration
- Prometheus Configuration
- In your custom_kube_prometheus_stack.yml, ensure the externalUrl for Prometheus is set correctly:
```bash
prometheus:
  prometheusSpec:
    externalUrl: "http://<LOAD_BALANCER_DNS>/prometheus"  # Replace with your LB DNS
    routePrefix: /prometheus  # Explicitly set the route prefix
```
## Step 2: Verify Ingress Configuration
- Alertmanager Configuration
```bash
alertmanager:
  alertmanagerSpec:
    externalUrl: "http://<LOAD_BALANCER_DNS>/alertmanager"  # Replace with your LB DNS
    routePrefix: /alertmanager  # Explicitly set the route prefix
```
```bash
    alb.ingress.kubernetes.io/healthcheck-path: /prometheus/-/healthy  # Prometheus health check
    alb.ingress.kubernetes.io/healthcheck-path: /alertmanager/-/healthy  # Alertmanager health check
    alb.ingress.kubernetes.io/healthcheck-path: /api/health  # Grafana health check
```
## Step 3: Verify Application Subpath Handling
- Prometheus Subpath Handling
```bash
prometheus:
  prometheusSpec:
    externalUrl: "http://<LOAD_BALANCER_DNS>/prometheus"
    routePrefix: /prometheus
```
- Alertmanager Subpath Handling
```bash
alertmanager:
  alertmanagerSpec:
    externalUrl: "http://<LOAD_BALANCER_DNS>/alertmanager"
    routePrefix: /alertmanager
```
Step 4: Test Access
After applying the above changes, test accessing the endpoints:

1. Prometheus: http://<LOAD_BALANCER_DNS>/prometheus
2. Alertmanager: http://<LOAD_BALANCER_DNS>/alertmanager
3. Grafana: http://<LOAD_BALANCER_DNS>/grafana

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
