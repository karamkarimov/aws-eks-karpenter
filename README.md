# aws-eks-karpenter

# EKS + Karpenter Terraform Setup

This repository bootstraps an **Amazon EKS** cluster using **Terraform**, with **Karpenter** configured for autoscaling across **x86 (amd64)** and **ARM64 (Graviton)** instance types. It supports **Spot** and **On-Demand** capacity for optimal performance and cost-efficiency.

---

## Code Structure
```
. 
├── main.tf # Main Terraform configuration 
├── variables.tf # Input variables definition 
├── terraform.tfvars # Input variable values (set your AWS account ID and region here) 
├── deployment.yaml # Example Kubernetes deployment to test Karpenter (targets x86) 
└── README.md # Project documentation
```

---

## Prerequisites

Ensure you have the following installed locally:
- [Terraform](https://developer.hashicorp.com/terraform/downloads) ≥ 1.3
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- AWS credentials with sufficient permissions (admin or similar)
- An existing S3 bucket for storing the Terraform backend state

---

## Deployment Steps

1. Clone the Repository

```bash
git clone https://github.com/your-org/eks-karpenter-terraform.git
cd eks-karpenter-terraform
```
## Configure aws

```bash
$ aws configure
AWS Access Key ID [None]: youraccountaccesskey
AWS Secret Access Key [None]: youraccountsecretkey
Default region name [None]: us-east-1
Default output format [None]:
```

## Create S3 bucket for state file

Create s3 bucket manually or with terraform and define bucket name in main.tf file s3 bucket part




2. Configure Variables
Update the terraform.tfvars file with your values:

```
region         = "us-east-1"
aws_account_id = "YOUR_AWS_ACCOUNT_ID"
cluster_name   = "karpenter"
```



3. Initialize Terraform

```bash
cd terraform-code/
terraform init
```
4. Apply the Terraform Plan

```bash
terraform plan
terraform apply -auto-approve
```
This will:

Create a VPC

Deploy an EKS cluster (v1.30)

Bootstrap Karpenter and configure it for autoscaling

5. Update Your Kubeconfig

```bash
aws eks --region us-east-1 update-kubeconfig --name karpenter
```
6. Check Karpenter is running:

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=karpenter
```
# Testing Karpenter Autoscaling
This project includes a sample deployment (deployment.yaml) that requests 1 vCPU on x86 nodes.

✏️ Example Deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inflate
spec:
  replicas: 0
  selector:
    matchLabels:
      app: inflate
  template:
    metadata:
      labels:
        app: inflate
    spec:
      nodeSelector:
        kubernetes.io/arch: amd64  # Change to arm64 for Graviton
      terminationGracePeriodSeconds: 0
      containers:
        - name: inflate
          image: public.ecr.aws/eks-distro/kubernetes/pause:3.7
          resources:
            requests:
              cpu: 1
```


# Deploy and Scale

```bash
kubectl apply -f deployment.yaml
kubectl scale deployment inflate --replicas=5
```

Karpenter will automatically provision a new node (Spot or On-Demand, x86 or ARM64 based on nodeSelector).

# Cleanup
To destroy all resources:

```bash
terraform destroy -auto-approve
```

# Notes
Only one security group and subnet set should have the karpenter.sh/discovery tag.

Modify the NodePool in main.tf to fine-tune CPU, memory, or architecture preferences.

Karpenter automatically scales down idle nodes using the consolidation policy.


## Resources

-  [Karpenter Documentation](https://karpenter.sh/docs/)
-  [Terraform AWS EKS Module](https://github.com/terraform-aws-modules/terraform-aws-eks)
-  [AWS EKS Best Practices Guide](https://aws.github.io/aws-eks-best-practices/)
