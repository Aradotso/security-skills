---
name: ikb42603-cloud-security-labs
description: AWS cloud computing security essentials lab exercises covering IAM, encryption, network security, and monitoring
triggers:
  - "how do I complete the cloud security labs"
  - "setup AWS IAM lab exercise"
  - "configure cloud encryption with KMS"
  - "implement AWS security group rules"
  - "setup CloudTrail and CloudWatch monitoring"
  - "create secure VPC isolation"
  - "configure AWS multi-factor authentication"
  - "troubleshoot AWS security configurations"
---

# IKB42603 Cloud Security Labs Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This project provides hands-on laboratory exercises for learning AWS cloud computing security essentials. It covers five core security domains: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

The labs are designed for educational purposes and follow a structured approach to implementing AWS security best practices using real AWS services.

## Repository Structure

The project organizes labs into separate markdown files:

```
IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/
├── README.md
├── Lab0_Environment_Setup.md
├── Lab1_Account_Security_and_IAM.md
├── Lab2_Secure_Isolation_and_Multitenancy.md
├── Lab3_Encryption_and_Key_Management.md
├── Lab4_Access_Control_and_Network_Security.md
└── Lab5_Monitoring_Logging_and_Incident_Detection.md
```

## Prerequisites

### Required Tools

1. **AWS Account** - Free tier eligible
2. **AWS CLI** - Command line interface for AWS services
3. **Git** - Version control
4. **Text Editor** - VS Code, Vim, or similar

### AWS CLI Installation

```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows (PowerShell)
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

### AWS CLI Configuration

```bash
# Configure AWS credentials
aws configure

# Enter your credentials when prompted
# AWS Access Key ID: ${AWS_ACCESS_KEY_ID}
# AWS Secret Access Key: ${AWS_SECRET_ACCESS_KEY}
# Default region: us-east-1
# Default output format: json
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-01

# Create access key for programmatic access
aws iam create-access-key --user-name lab-user-01

# Attach policy to user
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Creating IAM Groups and Roles

```bash
# Create IAM group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-01 \
  --group-name Developers

# Create IAM role with trust policy
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name EC2-S3-ReadOnly-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-ReadOnly-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### Enabling MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-mfa \
  --outfile qr-code.png \
  --bootstrap-method QRCodePNG

# Enable MFA for user (requires two consecutive authentication codes)
aws iam enable-mfa-device \
  --user-name lab-user-01 \
  --serial-number arn:aws:iam::${ACCOUNT_ID}:mfa/lab-user-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### Custom IAM Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadWriteSpecificBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-lab-bucket/*"
    },
    {
      "Sid": "AllowListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-lab-bucket"
    }
  ]
}
```

Apply custom policy:

```bash
aws iam put-user-policy \
  --user-name lab-user-01 \
  --policy-name S3BucketAccess \
  --policy-document file://custom-policy.json
```

## Lab 2: Secure Isolation and Multitenancy

### Creating VPC with Subnets

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=LabVPC}]'

# Store VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=LabVPC" \
  --query 'Vpcs[0].VpcId' \
  --output text)

# Create public subnet
aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'
```

### Internet Gateway and Route Tables

```bash
# Create and attach Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=LabIGW}]'

IGW_ID=$(aws ec2 describe-internet-gateways \
  --filters "Name=tag:Name,Values=LabIGW" \
  --query 'InternetGateways[0].InternetGatewayId' \
  --output text)

aws ec2 attach-internet-gateway \
  --vpc-id ${VPC_ID} \
  --internet-gateway-id ${IGW_ID}

# Create route table for public subnet
aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]'

RT_ID=$(aws ec2 describe-route-tables \
  --filters "Name=tag:Name,Values=PublicRouteTable" \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id ${RT_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${IGW_ID}

# Associate route table with public subnet
SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=PublicSubnet" \
  --query 'Subnets[0].SubnetId' \
  --output text)

aws ec2 associate-route-table \
  --route-table-id ${RT_ID} \
  --subnet-id ${SUBNET_ID}
```

### NAT Gateway for Private Subnet

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

EIP_ID=$(aws ec2 describe-addresses \
  --query 'Addresses[0].AllocationId' \
  --output text)

# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
  --subnet-id ${SUBNET_ID} \
  --allocation-id ${EIP_ID} \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=LabNAT}]'
```

## Lab 3: Encryption and Key Management

### Creating KMS Keys

```bash
# Create customer managed key
aws kms create-key \
  --description "Lab encryption key for S3" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

KEY_ID=$(aws kms list-keys --query 'Keys[0].KeyId' --output text)

# Create alias for the key
aws kms create-alias \
  --alias-name alias/lab-s3-key \
  --target-key-id ${KEY_ID}

# Get key details
aws kms describe-key --key-id alias/lab-s3-key
```

### S3 Bucket Encryption

```bash
# Create S3 bucket with encryption
aws s3api create-bucket \
  --bucket my-encrypted-lab-bucket-${RANDOM} \
  --region us-east-1

BUCKET_NAME=$(aws s3api list-buckets \
  --query 'Buckets[-1].Name' \
  --output text)

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket ${BUCKET_NAME} \
  --server-side-encryption-configuration '{
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "aws:kms",
          "KMSMasterKeyID": "'${KEY_ID}'"
        },
        "BucketKeyEnabled": true
      }
    ]
  }'

# Upload encrypted object
aws s3 cp document.txt s3://${BUCKET_NAME}/ \
  --sse aws:kms \
  --sse-kms-key-id ${KEY_ID}
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --size 10 \
  --encrypted \
  --kms-key-id ${KEY_ID} \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EncryptedVolume}]'

# Enable encryption by default for the region
aws ec2 enable-ebs-encryption-by-default
```

### RDS Encryption

```bash
# Create encrypted RDS instance
aws rds create-db-instance \
  --db-instance-identifier lab-encrypted-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password ${DB_PASSWORD} \
  --allocated-storage 20 \
  --storage-encrypted \
  --kms-key-id ${KEY_ID}
```

## Lab 4: Access Control and Network Security

### Security Groups

```bash
# Create security group for web server
aws ec2 create-security-group \
  --group-name WebServerSG \
  --description "Security group for web servers" \
  --vpc-id ${VPC_ID}

SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=WebServerSG" \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id ${SG_ID} \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id ${SG_ID} \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id ${SG_ID} \
  --protocol tcp \
  --port 22 \
  --cidr ${MY_IP}/32
```

### Network ACLs

```bash
# Create Network ACL
aws ec2 create-network-acl \
  --vpc-id ${VPC_ID} \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=CustomNACL}]'

NACL_ID=$(aws ec2 describe-network-acls \
  --filters "Name=tag:Name,Values=CustomNACL" \
  --query 'NetworkAcls[0].NetworkAclId' \
  --output text)

# Allow inbound HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id ${NACL_ID} \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow

# Allow outbound traffic
aws ec2 create-network-acl-entry \
  --network-acl-id ${NACL_ID} \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --egress true \
  --rule-action allow
```

### VPC Flow Logs

```bash
# Create CloudWatch Log Group
aws logs create-log-group --log-group-name /aws/vpc/flowlogs

# Create IAM role for Flow Logs
cat > flow-logs-trust.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name VPCFlowLogsRole \
  --assume-role-policy-document file://flow-logs-trust.json

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids ${VPC_ID} \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::${ACCOUNT_ID}:role/VPCFlowLogsRole
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-${ACCOUNT_ID}-${RANDOM} \
  --region us-east-1

TRAIL_BUCKET=$(aws s3api list-buckets \
  --query 'Buckets[-1].Name' \
  --output text)

# Apply bucket policy for CloudTrail
cat > cloudtrail-bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::${TRAIL_BUCKET}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::${TRAIL_BUCKET}/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket ${TRAIL_BUCKET} \
  --policy file://cloudtrail-bucket-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name lab-security-trail \
  --s3-bucket-name ${TRAIL_BUCKET}

# Start logging
aws cloudtrail start-logging --name lab-security-trail

# Enable log file validation
aws cloudtrail update-trail \
  --name lab-security-trail \
  --enable-log-file-validation
```

### CloudWatch Alarms

```bash
# Create SNS topic for alerts
aws sns create-topic --name SecurityAlerts

TOPIC_ARN=$(aws sns list-topics \
  --query 'Topics[?contains(TopicArn, `SecurityAlerts`)].TopicArn' \
  --output text)

# Subscribe email to topic
aws sns subscribe \
  --topic-arn ${TOPIC_ARN} \
  --protocol email \
  --notification-endpoint ${EMAIL_ADDRESS}

# Create CloudWatch alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICalls \
  --namespace AWS/CloudTrail \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions ${TOPIC_ARN}
```

### CloudWatch Logs Insights Queries

```bash
# Query CloudTrail logs for failed login attempts
aws logs start-query \
  --log-group-name /aws/cloudtrail \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string '
    fields @timestamp, userIdentity.principalId, eventName, errorCode
    | filter eventName = "ConsoleLogin" and errorMessage = "Failed authentication"
    | sort @timestamp desc
    | limit 20
  '
```

### GuardDuty Enablement

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

DETECTOR_ID=$(aws guardduty list-detectors \
  --query 'DetectorIds[0]' \
  --output text)

# Get findings
aws guardduty list-findings --detector-id ${DETECTOR_ID}

# Get finding details
FINDING_ID=$(aws guardduty list-findings \
  --detector-id ${DETECTOR_ID} \
  --query 'FindingIds[0]' \
  --output text)

aws guardduty get-findings \
  --detector-id ${DETECTOR_ID} \
  --finding-ids ${FINDING_ID}
```

## Common Patterns

### Lab Documentation Structure

Each lab should follow this structure in its markdown file:

```markdown
# Lab X: [Title]

## Objective
Brief description of what students will learn.

## Prerequisites
- AWS Account
- AWS CLI configured
- Specific permissions needed

## Tasks

### Task 1: [Task Name]
Step-by-step instructions with commands.

### Task 2: [Task Name]
More detailed steps.

## Verification
How to verify the lab was completed successfully.

## Screenshots
![Description](path/to/screenshot.png)

## Cleanup
Commands to remove resources and avoid charges.

## Reflection
Questions for students to consider.
```

### Resource Cleanup Script

```bash
#!/bin/bash
# cleanup.sh - Remove all lab resources

# Delete EC2 instances
aws ec2 terminate-instances --instance-ids $(aws ec2 describe-instances \
  --filters "Name=tag:Lab,Values=CloudSecurity" \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text)

# Delete Security Groups
for sg in $(aws ec2 describe-security-groups \
  --filters "Name=tag:Lab,Values=CloudSecurity" \
  --query 'SecurityGroups[].GroupId' \
  --output text); do
  aws ec2 delete-security-group --group-id $sg
done

# Delete S3 buckets
for bucket in $(aws s3api list-buckets \
  --query 'Buckets[?contains(Name, `lab`)].Name' \
  --output text); do
  aws s3 rm s3://$bucket --recursive
  aws s3api delete-bucket --bucket $bucket
done

# Delete IAM users
for user in $(aws iam list-users \
  --query 'Users[?contains(UserName, `lab`)].UserName' \
  --output text); do
  aws iam delete-user --user-name $user
done

# Delete CloudTrail trails
aws cloudtrail delete-trail --name lab-security-trail

# Disable GuardDuty
DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)
aws guardduty delete-detector --detector-id ${DETECTOR_ID}
```

## Troubleshooting

### AWS CLI Authentication Issues

```bash
# Verify credentials are configured
aws sts get-caller-identity

# Check if credentials are expired
aws sts get-session-token

# Reconfigure credentials
aws configure
```

### Permission Denied Errors

```bash
# Check effective permissions for current user
aws iam get-user

# Simulate policy to test permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::${ACCOUNT_ID}:user/lab-user-01 \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt
```

### VPC Connectivity Issues

```bash
# Check VPC configuration
aws ec2 describe-vpcs --vpc-ids ${VPC_ID}

# Verify Internet Gateway attachment
aws ec2 describe-internet-gateways \
  --filters "Name=attachment.vpc-id,Values=${VPC_ID}"

# Check route tables
aws ec2 describe-route-tables \
  --filters "Name=vpc-id,Values=${VPC_ID}"

# Verify security group rules
aws ec2 describe-security-groups --group-ids ${SG_ID}
```

### S3 Encryption Issues

```bash
# Verify bucket encryption settings
aws s3api get-bucket-encryption --bucket ${BUCKET_NAME}

# Check KMS key permissions
aws kms get-key-policy \
  --key-id ${KEY_ID} \
  --policy-name default

# Test encryption with a small file
echo "test" > test.txt
aws s3 cp test.txt s3://${BUCKET_NAME}/ \
  --sse aws:kms \
  --sse-kms-key-id ${KEY_ID} \
  --debug
```

### CloudTrail Not Logging

```bash
# Check trail status
aws cloudtrail get-trail-status --name lab-security-trail

# Verify S3 bucket policy
aws s3api get-bucket-policy --bucket ${TRAIL_BUCKET}

# Check CloudTrail event selectors
aws cloudtrail get-event-selectors --trail-name lab-security-trail

# Manually validate logs are being written
aws cloudtrail lookup-events --max-results 10
```

## Best Practices

1. **Always tag resources** with `Lab` and `Environment` tags for easy identification
2. **Use least privilege** when creating IAM policies
3. **Enable MFA** on all user accounts
4. **Encrypt sensitive data** at rest and in transit
5. **Enable logging** for all services (CloudTrail, VPC Flow Logs, S3 access logs)
6. **Clean up resources** after completing labs to avoid unnecessary charges
7. **Use CloudFormation** or Terraform for repeatable deployments
8. **Document everything** with screenshots and explanations
9. **Test security controls** by attempting unauthorized access
10. **Review costs regularly** using AWS Cost Explorer

## Environment Variables Reference

Store these in `.env` file (never commit to repository):

```bash
# AWS Credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-east-1

# Account Information
export ACCOUNT_ID=123456789012
export MY_IP=203.0.113.0

# Database Credentials
export DB_PASSWORD=SecurePassword123!

# Notification
export EMAIL_ADDRESS=student@example.com
```

Load environment variables:

```bash
source .env
```
