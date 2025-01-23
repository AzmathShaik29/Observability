# 🛠️  Installation & Configurations
## 📦 Step 1: Create EKS Cluster

### Prerequisites
- Download and Install AWS Cli - Please Refer this ("https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html") link
- Setup and configure AWS CLI using the `aws configure` command
- Install and configure eksctl using the steps mentioned [here]("https://eksctl.io/installation/")
- Install and configure kubectl as mentioned [here]("https://kubernetes.io/docs/tasks/tools/")
### After installation you should install the following to work on Prometheus, Grafan & Alertmanager. Either you can use PowerShell/GitBash/VScode
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

eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
