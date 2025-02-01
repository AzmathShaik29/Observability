# 🛠️  Installation & Configurations - With Ingress and Load Balancer
## 📦 Step 1: Create EKS Cluster

### Prerequisites
- Download and Install AWS CLI - Please Refer this ("https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html") link
- Setup and configure AWS CLI using the `aws configure` command
- Install and configure eksctl using the steps mentioned in this link or follow below steps ("https://eksctl.io/installation/")
- Install and configure kubectl as mentioned in this link or follow below steps ("https://kubernetes.io/docs/tasks/tools/")

## How to conffigure AWS CLI
**Login to your AWS Account (root/IAM user) through console and follow below steps to get the Access & Secret Keys:**

1. Go to Security Credentials
2. Click on Create access key - once you click the keys will generate and you should download it and keep it as safe as possible
![image](https://github.com/user-attachments/assets/c996bc93-ab4e-4ef5-a702-bb02e3e7ccb1)
![image](https://github.com/user-attachments/assets/36aec483-0d02-409b-9f52-9cb868b86c55)

4. Once the Access and Secret keys are generated then you are ready to configure them as shown in below image
  
![image](https://github.com/user-attachments/assets/5614fd31-9151-442c-8a87-f05fc7a44132)


**After installation you should install the following to work on Prometheus, Grafan & Alertmanager. Either you can use PowerShell/GitBash/VScode**

    1. eksctl
    2. kubectl
    3. helm

## eksctl

Using Chocolatey
- choco install eksctl

Or download and install manually
curl.exe -O "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Windows_amd64.zip"
Expand-Archive eksctl_Windows_amd64.zip -DestinationPath "C:\Program Files\eksctl"
Add to PATH: C:\Program Files\eksctl

Verify installation:
- eksctl version

## kubectl

Using Chocolatey
- choco install kubernetes-cli

Or download directly
curl.exe -LO "https://dl.k8s.io/release/v1.29.0/bin/windows/amd64/kubectl.exe"

Move to a directory in your PATH, for example:
Move-Item .\kubectl.exe -Destination 'C:\Program Files\kubectl\'

Verify installation:
- kubectl version --client

## helm
Using Chocolatey (recommended)
- choco install kubernetes-helm

Or using winget
winget install helm

Or download and install manually
Download the Windows binary from https://get.helm.sh/helm-v3.14.2-windows-amd64.zip
Extract it and add the folder to your PATH

Verify installation:
- helm version

## Once all the above installations has completed then you are ready to create the AWS EKS Cluster

```bash
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
```
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
```
```bash
eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

# Update ./kube/config file
aws eks update-kubeconfig --name observability
```

### 🧰 Step 2: Install kube-prometheus-stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 🚀 Step 3: Deploy the chart into a new namespace "monitoring"
```bash
kubectl create ns monitoring
```

**Here in the same folder I'm using the custom_kube_prometheus_stack.yml file for Alertmanager**
```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml
```

### Step 4: Install the AWS Load Balancer Controller
**Add the EKS chart repo**
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```
**Install the AWS Load Balancer Controller**
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=observability \
  --set serviceAccount.create=true
```

### Step 5: Create an ingress for your monitoring services. Create a file named monitoring-ingress.yaml in the same folder and apply the ingress configuration.
```bash
kubectl apply -f monitoring-ingress.yaml
```

- Verify the Installation
```bash
kubectl get all -n monitoring
```

- Checking ALB URL:
```bash
kubectl get ingress -n monitoring monitoring-ingress
``` 
- Check the status of the Ingress
```bash
kubectl get ingress -n monitoring
```
You should see an ADDRESS (the ALB URL). If not then check the Ingress Controller Logs
```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```
### Step 6: If you get AccessDenied error in the logs then follow below steps to create an IAM policy with below configuration.

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

### Step 7: After applying these changes:
1. Wait a few minutes for the ALB to be provisioned
2. Get the ALB URL:
```bash
kubectl get ingress -n monitoring monitoring-ingress
```

**You can then access your services at:**

- Prometheus: http://<ALB-URL>/prometheus
- Grafana: http://<ALB-URL>/grafana
- Alertmanager: http://<ALB-URL>/alertmanager


**NOTE:** Once you have performed all the steps then don't forget to destroy everything, otherwise AWS will charge you.

### 🧼 Step 8: Clean UP
- **Uninstall helm chart**:
```bash
helm uninstall monitoring --namespace monitoring
```
- **Delete namespace**:
```bash
kubectl delete ns monitoring
```
- **Delete Cluster & once you delete the cluster all the associated services will get deleted**:
```bash
eksctl delete cluster --name observability
```

