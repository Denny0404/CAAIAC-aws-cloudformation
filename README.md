
# Assignment 3: Deploying AWS EC2 Instance and RDS Instance using CloudFormation

**Author:** Denish Akbari  
**Student ID:** 8947486  
**Date:** [Insert Date]

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [AWS CLI & Tokens Setup](#aws-cli--tokens-setup)
4. [Project Structure](#project-structure)
5. [YAML Templates](#yaml-templates)
6. [Deployment Steps](#deployment-steps)
   - 6.1 [Deploy Networking Stack](#deploy-networking-stack)
   - 6.2 [Deploy EC2 Stack](#deploy-ec2-stack)
   - 6.3 [Deploy RDS Stack](#deploy-rds-stack)
7. [How to Find Resource Outputs](#how-to-find-resource-outputs)
8. [Screenshot Checklist](#screenshot-checklist)
9. [Cleanup (Optional)](#cleanup-optional)
10. [Troubleshooting & Common Errors](#troubleshooting--common-errors)

---

## Overview

This project demonstrates automated deployment of AWS resources for a typical cloud workload using CloudFormation and the AWS CLI. It covers:
- VPC with two public subnets (multi-AZ)
- Internet Gateway, Public Route Table
- EC2 Instance (with SSH access, custom key pair, user-chosen AMI/Instance Type)
- Public RDS MySQL instance, in a multi-AZ subnet group, with security groups

---

## Prerequisites

- AWS Account or AWS Academy account (with Learner Lab access)
- AWS CLI installed ([Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html))
- Region: `us-east-1`
- (Optional) [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation)
- EC2 Key Pair for SSH (can be created via AWS Console)

---

## AWS CLI & Tokens Setup

### 1. **Configure the AWS CLI with Keys and Token**

If using a regular AWS account:
```sh
aws configure
# Enter AWS Access Key ID, Secret Access Key, Default region name (us-east-1), and output format (json)
```

If using AWS Academy (Learner Lab):
1. Get **Access Key, Secret Key, and Session Token** from the Lab portal.
2. Run:
```sh
aws configure
# Enter Access Key, Secret Key, region (us-east-1), output (json)
```
3. Add your session token manually:
```sh
echo 'aws_session_token = <paste your AWS_SESSION_TOKEN here>' >> ~/.aws/credentials
```

### 2. **Test Your CLI Authentication**
```sh
aws sts get-caller-identity
# Should return your user/account info, not an error
```

### 3. **If You Need a Key Pair:**
- In AWS Console: EC2 → Network & Security → Key Pairs → Create key pair → Download .pem file
- Use the key pair **name** for the EC2 stack parameter.

---

## Project Structure

```
CAAIAC-aws-cloudformation/
├── networking.yaml      # VPC, subnets, IGW, route table
├── ec2.yaml             # EC2 instance & security group
├── rds.yaml             # RDS instance & security group
├── README.md            # This file
```

---

## YAML Templates

- `networking.yaml` – VPC, two subnets (us-east-1a & 1c), IGW, route table
- `ec2.yaml` – EC2 instance, SSH security group, key pair parameter
- `rds.yaml` – RDS instance (MySQL), security group, DB subnet group using both subnets

**All templates must be stored locally.**

---

## Deployment Steps

### 6.1 Deploy Networking Stack

```sh
aws cloudformation create-stack   --stack-name assignment3-networking   --template-body file://networking.yaml
```
- Wait for **CREATE_COMPLETE** in CloudFormation Console.
- Get Outputs: `VPCId`, `SubnetIdA`, `SubnetIdB`.

### 6.2 Deploy EC2 Stack

```sh
aws cloudformation create-stack   --stack-name assignment3-ec2   --template-body file://ec2.yaml   --parameters     ParameterKey=SubnetId,ParameterValue=<SubnetIdA>     ParameterKey=VpcId,ParameterValue=<VPCId>     ParameterKey=KeyName,ParameterValue=<YourKeyName>
```
Replace `<SubnetIdA>`, `<VPCId>`, and `<YourKeyName>` with your outputs and key pair.

### 6.3 Deploy RDS Stack

```sh
aws cloudformation create-stack   --stack-name assignment3-rds   --template-body file://rds.yaml   --parameters     ParameterKey=SubnetIdA,ParameterValue=<SubnetIdA>     ParameterKey=SubnetIdB,ParameterValue=<SubnetIdB>     ParameterKey=VpcId,ParameterValue=<VPCId>
```
Replace with actual outputs.

---

## How to Find Resource Outputs

- AWS Console → CloudFormation → Stacks → assignment3-networking/ec2/rds → **Outputs** tab
- Outputs include VPC, Subnet, EC2 Public IP, RDS Endpoint

---

## Screenshot Checklist

1. **VPC, Subnets, IGW:** VPC → Your VPCs, Subnets, Internet Gateways (with correct IDs)
2. **Route Table:** Route Tables (showing associations and 0.0.0.0/0 route)
3. **EC2 Instance:** EC2 → Instances (show instance ID, Name, Public IP)
4. **RDS Instance:** RDS → Databases (show DB name, endpoint, status)
5. **CloudFormation Stacks:** Events tab (creation steps) and Outputs tab (IDs, endpoints, etc.) for each stack
6. **CLI Terminal Output:** For every `aws cloudformation create-stack ...` command (shows StackId)

---

## Cleanup (Optional)

When done, remove all resources to avoid costs:

```sh
aws cloudformation delete-stack --stack-name assignment3-rds
aws cloudformation delete-stack --stack-name assignment3-ec2
aws cloudformation delete-stack --stack-name assignment3-networking
```

---

## Troubleshooting & Common Errors

- **SignatureDoesNotMatch or InvalidToken**: Double-check you added the `aws_session_token` if using AWS Academy.
- **AlreadyExistsException**: You can only create each stack name once at a time. Delete the stack before recreating.
- **Subnet Group Invalid**: Make sure you have two subnets in different AZs, and both are in the same VPC.
- **Public IP not assigned**: Make sure subnets have `MapPublicIpOnLaunch: true`.
- **Security Group Errors**: Use correct VPC ID for each resource's security group.

---

## Example CLI Session

```sh
# Configure AWS CLI (with session token for Academy)
aws configure
nano ~/.aws/credentials # Add aws_session_token = ... if needed

# Deploy Networking Stack
aws cloudformation create-stack --stack-name assignment3-networking --template-body file://networking.yaml
# Wait for CREATE_COMPLETE, note Outputs (VPCId, SubnetIdA, SubnetIdB)

# Deploy EC2 Stack
aws cloudformation create-stack --stack-name assignment3-ec2 --template-body file://ec2.yaml --parameters ParameterKey=SubnetId,ParameterValue=subnet-abc... ParameterKey=VpcId,ParameterValue=vpc-abc... ParameterKey=KeyName,ParameterValue=your-keypair

# Deploy RDS Stack
aws cloudformation create-stack --stack-name assignment3-rds --template-body file://rds.yaml --parameters ParameterKey=SubnetIdA,ParameterValue=subnet-abc... ParameterKey=SubnetIdB,ParameterValue=subnet-def... ParameterKey=VpcId,ParameterValue=vpc-abc...

# Cleanup (when done)
aws cloudformation delete-stack --stack-name assignment3-rds
aws cloudformation delete-stack --stack-name assignment3-ec2
aws cloudformation delete-stack --stack-name assignment3-networking
```

---

## Contact
For any issues, contact Denish Akbari or your instructor.
