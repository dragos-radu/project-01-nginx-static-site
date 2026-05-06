## Project 01 - Nginx Static Site

## Jira
DEVOPS-2

## Goal
Deploy a static site using Nginx.

## Steps
- install Nginx
- serve html

## Tech Stack
- AWS EC2
- Ubuntu
- Nginx
- Github
- Jira

## Status
Completed

---

# AWS EC2 Deployment Commands

## Configure AWS Region

```bash
export AWS_REGION=eu-central-1
```

---

## Create Key Pair

```bash
export KEY_NAME=devopsroad-key

aws ec2 create-key-pair \
    --region $AWS_REGION \
    --key-name $KEY_NAME \
    --query 'KeyMaterial' \
    --output text > ${KEY_NAME}.pem
```

---

## Set SSH Key Permissions

```bash
chmod 400 ${KEY_NAME}.pem
```

---

## Get Ubuntu 22.04 AMI ID

```bash
export AMI_ID=$(aws ssm get-parameters \
--region $AWS_REGION \
--names /aws/service/canonical/ubuntu/server/22.04/stable/current/amd64/hvm/ebs-gp2/ami-id \
--query "Parameters[0].Value" \
--output text)

echo $AMI_ID
```

---

## Create Security Group

```bash
export SECURITY_GROUP_ID=$(aws ec2 create-security-group \
    --region $AWS_REGION \
    --group-name devopsroad-sg \
    --description "DevOps Road Nginx Security Group" \
    --query 'GroupId' \
    --output text)

echo $SECURITY_GROUP_ID
```

---

## Open SSH Port (22)

```bash
aws ec2 authorize-security-group-ingress \
    --region $AWS_REGION \
    --group-id $SECURITY_GROUP_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

---

## Open HTTP Port (80)

```bash
aws ec2 authorize-security-group-ingress \
    --region $AWS_REGION \
    --group-id $SECURITY_GROUP_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

---

## Launch EC2 Instance

```bash
export INSTANCE_ID=$(aws ec2 run-instances \
    --region $AWS_REGION \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type t3.micro \
    --key-name $KEY_NAME \
    --security-group-ids $SECURITY_GROUP_ID \
    --query 'Instances[0].InstanceId' \
    --output text)

echo $INSTANCE_ID
```

---

## Wait Until Instance Is Running

```bash
aws ec2 wait instance-running \
    --region $AWS_REGION \
    --instance-ids $INSTANCE_ID
```

---

## Get Public IP

```bash
export PUBLIC_IP=$(aws ec2 describe-instances \
    --region $AWS_REGION \
    --instance-ids $INSTANCE_ID \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text)

echo $PUBLIC_IP
```

---

## Connect via SSH

```bash
ssh -i ${KEY_NAME}.pem ubuntu@$PUBLIC_IP
```

---

# Commands Executed on EC2

## Update Packages

```bash
sudo apt update
```

---

## Install Nginx

```bash
sudo apt install nginx -y
```

---

## Install Git

```bash
sudo apt install git -y
```

---

## Clone GitHub Repository

```bash
git clone https://github.com/YOUR_USERNAME/project-01-nginx-static-site.git
```

---

## Enter Repository

```bash
cd project-01-nginx-static-site
```

---

## Deploy HTML File

```bash
sudo cp app/index.html /var/www/html/index.html
```

---

## Verify Nginx Status

```bash
sudo systemctl status nginx
```

---

## Open Application

```text
http://PUBLIC_IP
```

---

# Cleanup

## Terminate EC2 Instance

```bash
aws ec2 terminate-instances \
    --region $AWS_REGION \
    --instance-ids $INSTANCE_ID
```

---

## Delete Security Group

```bash
aws ec2 delete-security-group \
    --region $AWS_REGION \
    --group-id $SECURITY_GROUP_ID
```

---

## Delete AWS Key Pair

```bash
aws ec2 delete-key-pair \
    --region $AWS_REGION \
    --key-name $KEY_NAME
```

---

## Delete Local PEM File

```bash
rm ${KEY_NAME}.pem
```