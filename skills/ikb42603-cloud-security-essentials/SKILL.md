---
name: ikb42603-cloud-security-essentials
description: Educational AWS cloud security labs covering IAM, VPC, encryption, KMS, monitoring, and incident detection
triggers:
  - how do I complete the cloud security labs
  - set up AWS IAM security lab
  - configure cloud encryption and key management
  - implement VPC network security
  - set up CloudWatch monitoring and logging
  - complete IKB42603 cloud security course
  - configure AWS security best practices
  - create isolated multi-tenant cloud environment
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603 Cloud Computing Security Essentials is an educational repository containing hands-on AWS security labs. The course covers five core areas: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

This skill helps you complete the lab exercises, configure AWS security services, and implement cloud security best practices.

## Repository Structure

The repository is organized into five weekly labs:

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

## Getting Started

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/<your-username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create initial structure
touch README.md
touch Lab0_Environment_Setup.md
touch Lab1_Account_Security_and_IAM.md
touch Lab2_Secure_Isolation_and_Multitenancy.md
touch Lab3_Encryption_and_Key_Management.md
touch Lab4_Access_Control_and_Network_Security.md
touch Lab5_Monitoring_Logging_and_Incident_Detection.md

# Initial commit
git add .
git commit -m "Initial lab setup"
git push origin main
```

### AWS CLI Configuration

```bash
# Configure AWS CLI with credentials
aws configure
# AWS Access Key ID: ${AWS_ACCESS_KEY_ID}
# AWS Secret Access Key: ${AWS_SECRET_ACCESS_KEY}
# Default region name: us-east-1
# Default output format: json

# Verify configuration
aws sts get-caller-identity
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create an IAM user
aws iam create-user --user-name lab-user-admin

# Create a user with specific tags
aws iam create-user --user-name lab-user-dev \
  --tags Key=Environment,Value=Development Key=Lab,Value=1

# List all users
aws iam list-users
```

### Configuring IAM Policies

```bash
# Create a custom policy (save as policy.json first)
cat > readonly-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name Lab1-ReadOnlyS3Policy \
  --policy-document file://readonly-policy.json

# Attach policy to user
aws iam attach-user-policy \
  --user-name lab-user-dev \
  --policy-arn arn:aws:iam::${AWS_ACCOUNT_ID}:policy/Lab1-ReadOnlyS3Policy
```

### Enabling MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-mfa-device \
  --outfile qr-code.png \
  --bootstrap-method QRCodePNG

# Enable MFA for user (after scanning QR code)
aws iam enable-mfa-device \
  --user-name lab-user-admin \
  --serial-number arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/lab-mfa-device \
  --authentication-code-1 <code1> \
  --authentication-code-2 <code2>
```

### Creating IAM Groups and Roles

```bash
# Create a group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-dev \
  --group-name Developers

# Create an IAM role for EC2
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
  --role-name Lab1-EC2-Role \
  --assume-role-policy-document file://trust-policy.json
```

## Lab 2: Secure Isolation and Multitenancy

### Creating VPCs

```bash
# Create a VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab2-VPC}]'

# Create subnets (public and private)
aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Lab2-Public-Subnet}]'

aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Lab2-Private-Subnet}]'
```

### Configuring Internet Gateway and NAT

```bash
# Create and attach Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab2-IGW}]'

aws ec2 attach-internet-gateway \
  --vpc-id ${VPC_ID} \
  --internet-gateway-id ${IGW_ID}

# Create NAT Gateway (requires Elastic IP)
aws ec2 allocate-address --domain vpc

aws ec2 create-nat-gateway \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --allocation-id ${EIP_ALLOCATION_ID} \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=Lab2-NAT}]'
```

### Route Tables

```bash
# Create route table for public subnet
aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Lab2-Public-RT}]'

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id ${PUBLIC_RT_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${IGW_ID}

# Associate route table with subnet
aws ec2 associate-route-table \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --route-table-id ${PUBLIC_RT_ID}
```

## Lab 3: Encryption and Key Management

### Creating KMS Keys

```bash
# Create a customer master key
aws kms create-key \
  --description "Lab3 encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create an alias for the key
aws kms create-alias \
  --alias-name alias/lab3-key \
  --target-key-id ${KEY_ID}

# List keys
aws kms list-keys
```

### Encrypting and Decrypting Data

```bash
# Encrypt plaintext
echo "sensitive data" > plaintext.txt

aws kms encrypt \
  --key-id alias/lab3-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt ciphertext
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode
```

### S3 Bucket Encryption

```bash
# Create encrypted S3 bucket
aws s3api create-bucket \
  --bucket lab3-encrypted-bucket-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket lab3-encrypted-bucket-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab3-key"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Upload encrypted object
aws s3 cp plaintext.txt s3://lab3-encrypted-bucket-${AWS_ACCOUNT_ID}/ \
  --sse aws:kms \
  --sse-kms-key-id alias/lab3-key
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id ${KEY_ID} \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=Lab3-Encrypted-Volume}]'
```

## Lab 4: Access Control and Network Security

### Security Groups

```bash
# Create a security group
aws ec2 create-security-group \
  --group-name Lab4-Web-SG \
  --description "Security group for web servers" \
  --vpc-id ${VPC_ID}

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
  --cidr ${YOUR_IP}/32
```

### Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id ${VPC_ID} \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Lab4-NACL}]'

# Add inbound rule (allow HTTP)
aws ec2 create-network-acl-entry \
  --network-acl-id ${NACL_ID} \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Add outbound rule
aws ec2 create-network-acl-entry \
  --network-acl-id ${NACL_ID} \
  --egress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow
```

### VPC Flow Logs

```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name /aws/vpc/lab4-flow-logs

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
  --role-name Lab4-VPC-FlowLogs-Role \
  --assume-role-policy-document file://flow-logs-trust.json

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids ${VPC_ID} \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/lab4-flow-logs \
  --deliver-logs-permission-arn arn:aws:iam::${AWS_ACCOUNT_ID}:role/Lab4-VPC-FlowLogs-Role
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket lab5-cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --region us-east-1

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
      "Resource": "arn:aws:s3:::lab5-cloudtrail-logs-${AWS_ACCOUNT_ID}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::lab5-cloudtrail-logs-${AWS_ACCOUNT_ID}/*",
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
  --bucket lab5-cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --policy file://cloudtrail-bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name Lab5-Trail \
  --s3-bucket-name lab5-cloudtrail-logs-${AWS_ACCOUNT_ID}

# Start logging
aws cloudtrail start-logging --name Lab5-Trail
```

### CloudWatch Alarms

```bash
# Create alarm for failed login attempts
aws cloudwatch put-metric-alarm \
  --alarm-name Lab5-FailedLoginAttempts \
  --alarm-description "Alert on multiple failed login attempts" \
  --metric-name FailedLoginAttempts \
  --namespace AWS/IAM \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1

# Create alarm for root account usage
aws cloudwatch put-metric-alarm \
  --alarm-name Lab5-RootAccountUsage \
  --alarm-description "Alert on root account usage" \
  --metric-name RootAccountUsage \
  --namespace AWS/CloudTrail \
  --statistic Sum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1
```

### CloudWatch Log Insights Queries

```bash
# Query VPC Flow Logs for rejected traffic
aws logs start-query \
  --log-group-name /aws/vpc/lab4-flow-logs \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, srcAddr, dstAddr, dstPort, action | filter action = "REJECT" | sort @timestamp desc | limit 20'

# Query CloudTrail for unauthorized API calls
aws logs start-query \
  --log-group-name /aws/cloudtrail/logs \
  --start-time $(date -u -d '24 hours ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, userIdentity.principalId, eventName, errorCode | filter errorCode = "UnauthorizedOperation" | sort @timestamp desc'
```

### AWS Config for Compliance

```bash
# Create configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=Lab5-Recorder,roleARN=arn:aws:iam::${AWS_ACCOUNT_ID}:role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig

# Start recording
aws configservice start-configuration-recorder \
  --configuration-recorder-name Lab5-Recorder

# Deploy managed config rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "s3-bucket-encryption-enabled",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
    }
  }'
```

## Common Patterns

### Lab Documentation Template

```markdown
# Lab X: [Lab Title]

## Objective
[What this lab aims to achieve]

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Environment
- AWS Region: us-east-1
- Services Used: IAM, VPC, EC2, etc.

## Implementation Steps

### Step 1: [Step Title]
[Description]

```bash
# Commands used
aws service command --parameters
```

**Screenshot:** ![Step 1](images/step1.png)

### Step 2: [Step Title]
[Continue...]

## Challenges Encountered
- Challenge 1 and how it was resolved
- Challenge 2 and solution

## Lessons Learned
- Key takeaway 1
- Key takeaway 2

## References
- [AWS Documentation](https://docs.aws.amazon.com)
```

### Git Workflow for Lab Submission

```bash
# Start working on a lab
git checkout -b lab1-iam-security

# Make changes and commit regularly
git add Lab1_Account_Security_and_IAM.md
git commit -m "Add IAM user creation steps"

git add images/iam-user-screenshot.png
git commit -m "Add IAM user creation screenshot"

# Complete the lab
git commit -m "Complete Lab 1: Account Security and IAM"

# Push to GitHub
git push origin lab1-iam-security

# Merge to main
git checkout main
git merge lab1-iam-security
git push origin main
```

## Troubleshooting

### AWS CLI Authentication Issues

```bash
# Verify credentials
aws sts get-caller-identity

# Check IAM user permissions
aws iam get-user --user-name ${IAM_USERNAME}

# List attached policies
aws iam list-attached-user-policies --user-name ${IAM_USERNAME}

# Reconfigure AWS CLI
aws configure
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=${VPC_ID}"

# Verify security group rules
aws ec2 describe-security-groups --group-ids ${SG_ID}

# Check network ACL rules
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=${VPC_ID}"

# Test connectivity with reachability analyzer
aws ec2 create-network-insights-path \
  --source ${SOURCE_INSTANCE_ID} \
  --destination ${DEST_INSTANCE_ID} \
  --protocol tcp \
  --destination-port 80
```

### KMS Permission Errors

```bash
# Check key policy
aws kms get-key-policy --key-id ${KEY_ID} --policy-name default

# List grants on key
aws kms list-grants --key-id ${KEY_ID}

# Verify IAM permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::${AWS_ACCOUNT_ID}:user/${USERNAME} \
  --action-names kms:Encrypt kms:Decrypt \
  --resource-arns arn:aws:kms:us-east-1:${AWS_ACCOUNT_ID}:key/${KEY_ID}
```

### CloudWatch Logs Not Appearing

```bash
# Verify log group exists
aws logs describe-log-groups --log-group-name-prefix /aws/

# Check CloudTrail status
aws cloudtrail get-trail-status --name Lab5-Trail

# Test log delivery
aws logs put-log-events \
  --log-group-name /aws/test \
  --log-stream-name test-stream \
  --log-events timestamp=$(date +%s000),message="Test log entry"
```

## Best Practices

1. **Always tag resources** with Lab number and purpose
2. **Document every step** with commands and screenshots
3. **Use environment variables** for account-specific values
4. **Enable MFA** on all IAM users
5. **Follow principle of least privilege** when assigning permissions
6. **Enable encryption** by default for all data stores
7. **Monitor costs** using AWS Cost Explorer
8. **Clean up resources** after completing labs to avoid charges
9. **Commit frequently** with descriptive messages
10. **Review security group rules** regularly

## Resource Cleanup

```bash
# Delete S3 buckets (empty first)
aws s3 rm s3://lab3-encrypted-bucket-${AWS_ACCOUNT_ID} --recursive
aws s3api delete-bucket --bucket lab3-encrypted-bucket-${AWS_ACCOUNT_ID}

# Delete EC2 instances
aws ec2 terminate-instances --instance-ids ${INSTANCE_ID}

# Delete VPC (delete dependencies first)
aws ec2 delete-vpc --vpc-id ${VPC_ID}

# Delete IAM users
aws iam delete-user --user-name lab-user-dev

# Delete KMS keys (schedule deletion)
aws kms schedule-key-deletion --key-id ${KEY_ID} --pending-window-in-days 7

# Stop CloudTrail logging
aws cloudtrail stop-logging --name Lab5-Trail
aws cloudtrail delete-trail --name Lab5-Trail
```
