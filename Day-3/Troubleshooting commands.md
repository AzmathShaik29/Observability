- When I'm working on Ingress with Load Balancer I got stuck with accessing the Prometheus, Grafana and Alertmanager dashboards with errors. How I overcome with those errors and how I troubleshooted those errors.
### Below are the few troubleshooting commands for quick reference

### AccessDenied Error: Create an IAM policy with below configuration.

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
