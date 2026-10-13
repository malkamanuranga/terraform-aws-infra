# 🚀 Terraform AWS Infrastructure

> Infrastructure-as-Code (IaC) project that provisions a complete AWS environment — VPC, public subnet, internet gateway, route table, security group, and EC2 instance — with a single command.

[![Terraform](https://img.shields.io/badge/Terraform-%3E%3D1.5-7B42BC?logo=terraform&logoColor=white)](https://www.terraform.io/)
[![AWS](https://img.shields.io/badge/AWS-ap--south--1-FF9900?logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

---

## 📖 Overview

This project demonstrates **Infrastructure as Code** using Terraform to replace manual "ClickOps" with reproducible, version-controlled AWS infrastructure. Every resource is defined in code and can be created or destroyed with a single command.

**Region:** `ap-south-1` (Mumbai)
**Provider:** HashiCorp AWS `~> 5.0`
**Instance:** Ubuntu 22.04 LTS (`t2.micro`)

---

## 🏗️ Architecture

```
                        ┌─────────────────────────────────────────────┐
                        │              AWS Cloud (ap-south-1)         │
                        │                                             │
                        │   ┌─────────────────────────────────────┐   │
                        │   │        VPC  10.0.0.0/16             │   │
                        │   │                                     │   │
                        │   │   ┌─────────────────────────────┐   │   │
                        │   │   │  Public Subnet 10.0.1.0/24  │   │   │
                        │   │   │      (ap-south-1a)          │   │   │
                        │   │   │                             │   │   │
                        │   │   │      ┌──────────────┐       │   │   │
                        │   │   │      │  EC2 Instance│       │   │   │
                        │   │   │      │  Ubuntu 22.04│       │   │   │
                        │   │   │      │  SG: 22/80/443       │   │   │
                        │   │   │      └──────┬───────┘       │   │   │
                        │   │   └─────────────┼───────────────┘   │   │
                        │   │                 │                   │   │
                        │   │       Route Table 0.0.0.0/0         │   │
                        │   │                 │                   │   │
                        │   │        ┌────────▼────────┐          │   │
                        │   │        │ Internet Gateway│          │   │
                        │   │        └────────┬────────┘          │   │
                        │   └─────────────────┼───────────────────┘   │
                        └─────────────────────┼───────────────────────┘
                                              │
                                        ┌─────▼─────┐
                                        │  Internet │
                                        └───────────┘
```

---

## 📦 Resources Created

| # | Resource | Type | Description |
|---|----------|------|-------------|
| 1 | `aws_vpc` | Network | Isolated virtual network (`10.0.0.0/16`) |
| 2 | `aws_subnet` | Network | Public subnet with auto-assign public IP |
| 3 | `aws_internet_gateway` | Network | Enables internet access for the VPC |
| 4 | `aws_route_table` | Network | Routes `0.0.0.0/0` traffic to the IGW |
| 5 | `aws_route_table_association` | Network | Links subnet to route table |
| 6 | `aws_security_group` | Security | Firewall rules for SSH (22), HTTP (80), HTTPS (443) |
| 7 | `aws_instance` | Compute | Ubuntu 22.04 LTS EC2 instance |

---

## 📁 Project Structure

```
terraform-aws-infra/
├── main.tf            # All AWS resources
├── .gitignore         # Excludes state files and .terraform/
└── README.md          # You are here
```

---

## ✅ Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) `>= 1.5`
- [AWS CLI](https://aws.amazon.com/cli/) configured with valid credentials
  ```bash
  aws configure
  ```
- An existing AWS key pair for SSH access
  ```bash
  aws ec2 create-key-pair --key-name devops-key \
    --region ap-south-1 --query 'KeyMaterial' \
    --output text > devops-key.pem
  chmod 400 devops-key.pem
  ```

---

## 🚀 Usage

### 1. Clone the repository
```bash
git clone https://github.com/malkamanuranga/terraform-aws-infra.git
cd terraform-aws-infra
```

### 2. Initialize Terraform
Downloads the AWS provider plugin.
```bash
terraform init
```

### 3. Preview changes
```bash
terraform plan
```

### 4. Provision infrastructure
```bash
terraform apply
```
Type `yes` when prompted. Resources are created in ~30 seconds.

### 5. Get the EC2 public IP
```bash
aws ec2 describe-instances --region ap-south-1 \
  --filters "Name=tag:Name,Values=devops-ec2-instance" \
  --query 'Reservations[].Instances[].PublicIpAddress' --output text
```

### 6. SSH into the instance
```bash
ssh -i ~/devops-key.pem ubuntu@<PUBLIC_IP>
```

### 7. Destroy everything
```bash
terraform destroy
```
Type `yes` to confirm.

---

## 🔐 Security Notes

- State files (`*.tfstate`) are **never committed** — they contain sensitive data.
- The Security Group is intentionally permissive (`0.0.0.0/0`) for demo purposes. **In production**, restrict SSH to known CIDR ranges.
- Use IAM users with least-privilege permissions — **never** root access keys.

---

## 🧹 Clean Up

This project creates billable AWS resources. Always run:
```bash
terraform destroy
```

---

## 🛠️ Tech Stack

- **Terraform** — Infrastructure as Code
- **AWS** — Cloud provider
- **HCL** — HashiCorp Configuration Language

---

## 📚 What I Learned

- Writing HCL to define networking primitives (VPC, subnets, IGW, route tables)
- Managing resource dependencies with implicit references (`aws_vpc.my_vpc.id`)
- The Terraform lifecycle: `init → plan → apply → destroy`
- Idempotent, version-controlled infrastructure

---

## 👤 Author

**Malka Manuranga**
- GitHub: [@malkamanuranga](https://github.com/malkamanuranga)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
