# 🛠️  Installation & Configurations
## 📦 Step 1: Create EKS Cluster

### Prerequisites
- Download and Install AWS Cli - Please Refer this ("https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html") link
- Setup and configure AWS CLI using the `aws configure` command
- Install and configure eksctl using the steps mentioned in this link or follow below steps ("https://eksctl.io/installation/")
- Install and configure kubectl as mentioned in this link or follow below steps ("https://kubernetes.io/docs/tasks/tools/")

## How to conffigure AWS CLI
**Login to your AWS Account (root/IAM user) through console and follow below steps to get the Access & Secret Keys:**

1. Go to Security Credentials
2. Click on Create access key - once you click the keys will generate and you should download it and keep it as safe as possible
![image](https://github.com/user-attachments/assets/c996bc93-ab4e-4ef5-a702-bb02e3e7ccb1)
![image](https://github.com/user-attachments/assets/36aec483-0d02-409b-9f52-9cb868b86c55)
3. Once Access and Secret keys are created configure them as shown in below image
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
```bash
# Here in the same folder I'm using the custom_kube_prometheus_stack.yml file for Alertmanager

helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml
```

### ✅ Step 4: Verify the Installation
```bash
kubectl get all -n monitoring
```
- **Prometheus UI**:
```bash
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
```

**NOTE:** If you are using an EC2 Instance or Cloud VM, you need to pass `--address 0.0.0.0` to the above command. Then you can access the UI on <instance-ip:port>

**For e.g:** ```bash kubectl port-forward service/prometheus-operated -n monitoring 9090:9090 --address 0.0.0.0 ```
- **Grafana UI there is a password**:
- password is `prom-operator`
```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
```
- **Alertmanager UI**:
```bash
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
```
**NOTE:** Once you have performed all the steps then don't forget to destroy everything, otherwise AWS will charge you.

### 🧼 Step 5: Clean UP
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
