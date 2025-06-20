### What is CloudFormation?

CloudFormation is an open-source infrastructure as code (IaC) service provided by AWS that allows you to create, manage, and deploy production-ready environments. CloudFormation codifies AWS resource configurations into declarative YAML or JSON templates. It enables you to manage both existing AWS services and custom resources in a secure, repeatable, and automated way.

![1](https://github.com/stevymonkam/CloudFormation-3-tiers/blob/main/images/awscloudformation.PNG)

In this tutorial, I will deploy a three-tier application in AWS using CloudFormation.

![2](https://github.com/DhruvinSoni30/Terraform-AWS-3tier-Architecture/blob/main/2.png)





# 📦 AWS CloudFormation 3-Tier Architecture with Nested Stacks

This project provisions a complete 3-tier infrastructure on AWS using **CloudFormation nested stacks**. It separates network, security, compute, and database resources into modular templates for better scalability, maintenance, and team collaboration.

---



## 📂 Repository Example Structure

```
.
infra-cloudformation/
├── .github/
│   └── workflows/
│       ├── deploy-dev.yml
│       └── deploy-prod.yml
├── templates/
│   ├── network.yaml
│   ├── compute.yaml
│   ├── database.yaml
│   └── master-template.yaml
├── parameters/
│   ├── dev-parameters.json
│   └── prod-parameters.json
└── README.md
```

---

## 📌 Project Overview

The infrastructure includes:

* 📡 **VPC** with public and private subnets
* 🛡️ **Security Groups** for ALB, EC2 instances, and RDS
* 🌐 **Application Load Balancer (ALB)**
* 💻 **Auto Scaling Group of EC2 Instances**
* 📄 **RDS MySQL Database** with subnet groups for high availability

---

## 📂 Template Structure

| 📁 S3 Location                                                 | 📄 Template           | 🔹 Description                                     |
| :------------------------------------------------------------- | :-------------------- | :------------------------------------------------- |
| `s3://terraform-backend-stevy01/templates/network.yaml`        | `network.yaml`        | Creates VPC, public and private subnets            |
| `s3://terraform-backend-stevy01/templates/securitygroups.yaml` | `securitygroups.yaml` | Creates security groups for ALB, EC2, and RDS      |
| `s3://terraform-backend-stevy01/templates/alb.yaml`            | `alb.yaml`            | Creates Application Load Balancer and Target Group |
| `s3://terraform-backend-stevy01/templates/compute.yaml`        | `compute.yaml`        | Creates Launch Template and Auto Scaling Group     |
| `s3://terraform-backend-stevy01/templates/database.yaml`       | `database.yaml`       | Creates DB Subnet Group and RDS instance           |
| local                                                          | `master.yaml`         | Master stack orchestrating all nested stacks       |

---

## 📦 Storing Templates in S3

Before deploying, upload all nested templates to your designated S3 bucket:

```bash
aws s3 cp network.yaml s3://terraform-backend-stevy01/templates/
aws s3 cp securitygroups.yaml s3://terraform-backend-stevy01/templates/
aws s3 cp alb.yaml s3://terraform-backend-stevy01/templates/
aws s3 cp compute.yaml s3://terraform-backend-stevy01/templates/
aws s3 cp database.yaml s3://terraform-backend-stevy01/templates/
```

The `master.yaml` file references these templates via their `TemplateURL` in S3.

---

## 📜 Template Contents

### ✅ `network.yaml`

Creates the VPC, public subnets, and private subnets for RDS.
**Outputs:** VPC ID, Public Subnets, and Database Subnets.

### ✅ `securitygroups.yaml`

Creates security groups for:

* Application Load Balancer (allow HTTP)
* EC2 instances (allow SSH, HTTP, HTTPS)
* RDS (allow MySQL connections)
  **Outputs:** Security Group IDs for ALB, EC2, and RDS.

### ✅ `alb.yaml`

Creates:

* Application Load Balancer
* Target Group (for EC2 instances)
* Listener for forwarding traffic
  **Outputs:** ALB DNS name and Target Group ARN.

### ✅ `compute.yaml`

Creates:

* EC2 Launch Template using a specified Key Pair
* Auto Scaling Group attached to the Load Balancer’s Target Group

### ✅ `database.yaml`

Creates:

* DB Subnet Group with two private subnets
* RDS MySQL instance in Multi-AZ mode
  **Outputs:** RDS Endpoint and Instance ID.

---

## 📦 `master.yaml`

Orchestrates the creation order of nested stacks:

1. **NetworkStack**
2. **SecurityGroupsStack**
3. **ALBStack**
4. **ComputeStack**
5. **DatabaseStack**

It links parameters and outputs between child stacks to pass required values.

---

## 🚀 Deployment Instructions

Initialize and deploy the master stack from your local machine or CI/CD pipeline:

```bash
aws cloudformation deploy \
  --template-file master.yaml \
  --stack-name three-tier-dev \
  --parameter-overrides KeyName=my-keypair \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## 🎭 Clean Modular Design

Using nested stacks improves code readability, modularity, and allows different team members to work concurrently on different infrastructure layers (network, compute, database, etc.).

---

## 🌐 CI/CD Deployment with GitHub Actions

A CI/CD pipeline is integrated using **GitHub Actions**. It automates the deployment process whenever code is pushed to specific branches. Each environment (dev, prod) has its own workflow file.

Example of a development deployment workflow (`.github/workflows/deploy-dev.yml`):

```yaml
name: Deploy Dev Stack

on:
  push:
    branches:
      - develop

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_DEV }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_DEV }}
          aws-region: us-east-1

      - name: Deploy CloudFormation Stack
        run: |
          aws cloudformation deploy \
            --template-file master.yaml \
            --stack-name three-tier-dev \
            --parameter-overrides KeyName=my-keypair \
            --capabilities CAPABILITY_NAMED_IAM
```

**Secrets and credentials are managed securely via GitHub repository secrets** and never hardcoded in the workflow or code.

This ensures that each environment remains isolated, and deployments are automatic and reproducible.

---

## ✅ Final Expected Result

* ALB accessible via DNS output
* Auto Scaling EC2 instances behind ALB
* RDS MySQL instance in private subnets
* VPC with public and private subnets properly isolated
* Configured security groups for each component

---

## 📎 Notes

* Update AMI IDs in `compute.yaml` as per your AWS region.
* Replace the default RDS password in `database.yaml` with a secure Secrets Manager parameter in production.
* To remove stacks stuck in `ROLLBACK_COMPLETE` state:

```bash
aws cloudformation delete-stack --stack-name three-tier-dev
```

---


## 📌 Conclusion

This project provides a clean, modular, production-ready AWS infrastructure as code foundation for 3-tier web applications, fully orchestrated with CloudFormation nested stacks and managed in S3.

![1](https://github.com/stevymonkam/CloudFormation-3-tiers/blob/main/images/Screenshot%20(559).png)
![1](https://github.com/stevymonkam/CloudFormation-3-tiers/blob/main/images/Screenshot%20(561).png)
![1](https://github.com/stevymonkam/CloudFormation-3-tiers/blob/main/images/Screenshot%20(563).png)
![1](https://github.com/stevymonkam/CloudFormation-3-tiers/blob/main/images/Screenshot%20(564).png)
![1](https://github.com/stevymonkam/CloudFormation-3-tiers/blob/main/images/Screenshot%20(565).png)
