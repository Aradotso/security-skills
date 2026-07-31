---
name: ikb42603-cloud-security-essentials
description: Educational AWS cloud security labs covering IAM, encryption, network security, monitoring, and incident detection for IKB42603 course
triggers:
  - "help me with cloud security lab"
  - "setup AWS IAM security lab"
  - "configure AWS encryption and KMS"
  - "implement cloud security monitoring"
  - "setup VPC and security groups"
  - "work on cloud computing security essentials"
  - "complete IKB42603 lab"
  - "AWS security best practices tutorial"
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603 Cloud Computing Security Essentials is an educational repository containing structured laboratory exercises focused on AWS cloud security fundamentals. The course covers five core security domains: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

This skill helps AI agents guide students through AWS security implementations, lab documentation, and best practices for cloud security essentials.

## Repository Structure

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

## Setup and Installation

### Prerequisites

- AWS Account (Free Tier eligible)
- AWS CLI installed and configured
- Git installed
- Basic understanding of cloud computing concepts

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Configure AWS CLI
aws configure
# AWS Access Key ID: ${AWS_ACCESS_KEY_ID}
# AWS Secret Access Key: ${AWS_SECRET_ACCESS_KEY}
# Default region name: us-east-1
# Default output format: json

# Verify AWS configuration
aws sts get-caller-identity
```

## Lab 1: Account Security and IAM

### Creating IAM Users with Least Privilege

```bash
# Create an IAM user
aws iam create-user --user-name lab-user-readonly

# Attach read-only policy
aws iam attach-user-policy \
  --user-name lab-user-readonly \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Create access key for programmatic access
aws iam create-access-key --user-name lab-user-readonly

# Enable MFA (requires virtual MFA device serial)
aws iam enable-mfa-device \
  --user-name lab-user-readonly \
  --serial-number arn:aws:iam::${ACCOUNT_ID}:mfa/lab-user-readonly \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### Creating Custom IAM Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::lab-bucket-${ACCOUNT_ID}",
        "arn:aws:s3:::lab-bucket-${ACCOUNT_ID}/*"
      ]
    }
  ]
}
```

```bash
# Create custom policy
aws iam create-policy \
  --policy-name S3ReadOnlyLabPolicy \
  --policy-document file://s3-readonly-policy.json

# Attach custom policy to user
aws iam attach-user-policy \
  --user-name lab-user-readonly \
  --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/S3ReadOnlyLabPolicy
```

### IAM Roles for EC2

```bash
# Create trust policy for EC2
cat > ec2-trust-policy.json <<EOF
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

# Create IAM role
aws iam create-role \
  --role-name EC2-S3-ReadOnly-Role \
  --assume-role-policy-document file://ec2-trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-ReadOnly-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-ReadOnly-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-ReadOnly-Profile \
  --role-name EC2-S3-ReadOnly-Role
```

## Lab 2: Secure Isolation and Multitenancy

### Creating VPC with Subnets

```bash
# Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]' \
  --query 'Vpc.VpcId' \
  --output text)

# Create public subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-public-subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# Create private subnet
PRIVATE_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-private-subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# Create internet gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]' \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
```

### Security Groups for Isolation

```bash
# Create security group for web tier
WEB_SG_ID=$(aws ec2 create-security-group \
  --group-name web-tier-sg \
  --description "Security group for web tier" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

# Allow HTTP/HTTPS from internet
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Create security group for database tier
DB_SG_ID=$(aws ec2 create-security-group \
  --group-name db-tier-sg \
  --description "Security group for database tier" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

# Allow MySQL only from web tier
aws ec2 authorize-security-group-ingress \
  --group-id $DB_SG_ID \
  --protocol tcp \
  --port 3306 \
  --source-group $WEB_SG_ID
```

## Lab 3: Encryption and Key Management

### AWS KMS Key Creation and Management

```bash
# Create KMS key
KEY_ID=$(aws kms create-key \
  --description "Lab encryption key for S3" \
  --query 'KeyMetadata.KeyId' \
  --output text)

# Create key alias
aws kms create-alias \
  --alias-name alias/lab-s3-encryption-key \
  --target-key-id $KEY_ID

# Get key policy
aws kms get-key-policy \
  --key-id $KEY_ID \
  --policy-name default
```

### S3 Bucket Encryption

```bash
# Create S3 bucket with encryption
aws s3api create-bucket \
  --bucket lab-encrypted-bucket-${ACCOUNT_ID} \
  --region us-east-1

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket lab-encrypted-bucket-${ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "aws:kms",
          "KMSMasterKeyID": "'$KEY_ID'"
        },
        "BucketKeyEnabled": true
      }
    ]
  }'

# Verify encryption configuration
aws s3api get-bucket-encryption \
  --bucket lab-encrypted-bucket-${ACCOUNT_ID}
```

### Encrypting and Decrypting Data

```bash
# Encrypt a file
echo "Sensitive lab data" > plaintext.txt

aws kms encrypt \
  --key-id $KEY_ID \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt the file
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
VOLUME_ID=$(aws ec2 create-volume \
  --size 10 \
  --encrypted \
  --kms-key-id $KEY_ID \
  --availability-zone us-east-1a \
  --volume-type gp3 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=lab-encrypted-volume}]' \
  --query 'VolumeId' \
  --output text)

# Verify encryption status
aws ec2 describe-volumes \
  --volume-ids $VOLUME_ID \
  --query 'Volumes[0].Encrypted'
```

## Lab 4: Access Control and Network Security

### Network ACLs

```bash
# Create Network ACL
NACL_ID=$(aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=lab-nacl}]' \
  --query 'NetworkAcl.NetworkAclId' \
  --output text)

# Add inbound rule allowing HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Add outbound rule allowing all traffic
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --egress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Associate NACL with subnet
aws ec2 replace-network-acl-association \
  --association-id $(aws ec2 describe-network-acls \
    --filters "Name=association.subnet-id,Values=$PUBLIC_SUBNET_ID" \
    --query 'NetworkAcls[0].Associations[0].NetworkAclAssociationId' \
    --output text) \
  --network-acl-id $NACL_ID
```

### VPC Flow Logs

```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name /aws/vpc/flowlogs

# Create IAM role for VPC Flow Logs
cat > vpc-flow-logs-trust-policy.json <<EOF
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

FLOW_LOGS_ROLE_ARN=$(aws iam create-role \
  --role-name VPCFlowLogsRole \
  --assume-role-policy-document file://vpc-flow-logs-trust-policy.json \
  --query 'Role.Arn' \
  --output text)

# Attach policy
aws iam attach-role-policy \
  --role-name VPCFlowLogsRole \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids $VPC_ID \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn $FLOW_LOGS_ROLE_ARN
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-${ACCOUNT_ID} \
  --region us-east-1

# Apply bucket policy
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
      "Resource": "arn:aws:s3:::cloudtrail-logs-${ACCOUNT_ID}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::cloudtrail-logs-${ACCOUNT_ID}/*",
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
  --bucket cloudtrail-logs-${ACCOUNT_ID} \
  --policy file://cloudtrail-bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name lab-trail \
  --s3-bucket-name cloudtrail-logs-${ACCOUNT_ID}

# Start logging
aws cloudtrail start-logging --name lab-trail

# Verify trail status
aws cloudtrail get-trail-status --name lab-trail
```

### CloudWatch Alarms for Security

```bash
# Create CloudWatch alarm for failed console sign-ins
aws cloudwatch put-metric-alarm \
  --alarm-name failed-console-signin-alarm \
  --alarm-description "Alarm for failed console sign-in attempts" \
  --metric-name UserErrorRate \
  --namespace AWS/CloudTrail \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1

# Create SNS topic for alerts
TOPIC_ARN=$(aws sns create-topic \
  --name security-alerts \
  --query 'TopicArn' \
  --output text)

# Subscribe email to topic
aws sns subscribe \
  --topic-arn $TOPIC_ARN \
  --protocol email \
  --notification-endpoint ${NOTIFICATION_EMAIL}

# Add SNS action to alarm
aws cloudwatch put-metric-alarm \
  --alarm-name failed-console-signin-alarm \
  --alarm-actions $TOPIC_ARN \
  --metric-name UserErrorRate \
  --namespace AWS/CloudTrail \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1
```

### CloudWatch Logs Insights Queries

```bash
# Query for unauthorized API calls
aws logs start-query \
  --log-group-name /aws/cloudtrail \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, eventName, errorCode, userIdentity.userName
| filter errorCode like /UnauthorizedOperation|AccessDenied/
| sort @timestamp desc
| limit 20'

# Query for IAM policy changes
aws logs start-query \
  --log-group-name /aws/cloudtrail \
  --start-time $(date -u -d '24 hours ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, eventName, userIdentity.userName, requestParameters
| filter eventName like /Put|Delete|Create|Update/ and eventSource = "iam.amazonaws.com"
| sort @timestamp desc'
```

### Security Hub Integration

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Enable CIS AWS Foundations Benchmark
aws securityhub batch-enable-standards \
  --standards-subscription-requests '[{"StandardsArn":"arn:aws:securityhub:us-east-1::standards/cis-aws-foundations-benchmark/v/1.2.0"}]'

# Get findings
aws securityhub get-findings \
  --filters '{"SeverityLabel": [{"Value": "CRITICAL", "Comparison": "EQUALS"}]}' \
  --max-items 10
```

## Common Lab Patterns

### Lab Documentation Template

Each lab should follow this structure in its markdown file:

```markdown
# Lab X: [Lab Title]

## Objective
[Clear statement of what students will learn]

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Prerequisites
- AWS Account configured
- AWS CLI installed
- Basic knowledge of [relevant topics]

## Step-by-Step Implementation

### Task 1: [Task Name]

#### Commands
```bash
# Command with explanation
aws [service] [action]
```

#### Screenshot
![Description](path/to/screenshot.png)

#### Explanation
[Detailed explanation of what this accomplishes]

### Task 2: [Task Name]
[Continue pattern]

## Challenges Encountered
- Challenge 1 and solution
- Challenge 2 and solution

## Lessons Learned
- Key takeaway 1
- Key takeaway 2

## References
- [Relevant AWS documentation]
- [Additional resources]
```

### Git Workflow for Labs

```bash
# Start a new lab
git checkout -b lab-X-feature

# Make changes and commit regularly
git add Lab_X_[Topic].md screenshots/
git commit -m "Complete Task 1: [Description]"

# Push to repository
git push origin lab-X-feature

# Merge to main when complete
git checkout main
git merge lab-X-feature
git push origin main
```

### Environment Variables Setup

```bash
# Create .env file (DO NOT COMMIT)
cat > .env <<EOF
AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
AWS_DEFAULT_REGION=us-east-1
ACCOUNT_ID=${ACCOUNT_ID}
NOTIFICATION_EMAIL=${NOTIFICATION_EMAIL}
EOF

# Add to .gitignore
echo ".env" >> .gitignore
echo "*.pem" >> .gitignore
echo "plaintext.txt" >> .gitignore
```

## Troubleshooting

### AWS CLI Authentication Issues

```bash
# Verify credentials
aws sts get-caller-identity

# Check configured profile
aws configure list

# Use specific profile
aws s3 ls --profile lab-profile
```

### IAM Permission Errors

```bash
# Simulate policy to test permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::${ACCOUNT_ID}:user/lab-user \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::lab-bucket-${ACCOUNT_ID}/*

# Check attached policies
aws iam list-attached-user-policies --user-name lab-user
aws iam list-user-policies --user-name lab-user
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID"

# Check security group rules
aws ec2 describe-security-groups --group-ids $WEB_SG_ID

# Check NACL rules
aws ec2 describe-network-acls --network-acl-ids $NACL_ID

# Test network connectivity from EC2
# SSH into instance first
curl -v http://example.com
ping 8.8.8.8
traceroute 8.8.8.8
```

### CloudTrail Not Logging

```bash
# Check trail status
aws cloudtrail get-trail-status --name lab-trail

# Verify S3 bucket policy
aws s3api get-bucket-policy --bucket cloudtrail-logs-${ACCOUNT_ID}

# Check trail configuration
aws cloudtrail describe-trails --trail-name-list lab-trail
```

### KMS Key Access Issues

```bash
# Check key policy
aws kms get-key-policy --key-id $KEY_ID --policy-name default

# List grants on key
aws kms list-grants --key-id $KEY_ID

# Test encryption capability
echo "test" | aws kms encrypt --key-id $KEY_ID --plaintext fileb:///dev/stdin
```

## Best Practices for Lab Completion

1. **Documentation**: Always document commands with explanations
2. **Screenshots**: Include before/after screenshots for verification
3. **Clean Up**: Delete resources after lab completion to avoid charges
4. **Version Control**: Commit after each major task completion
5. **Security**: Never commit credentials, use environment variables
6. **Testing**: Verify each step works before proceeding
7. **Cost Management**: Use AWS Free Tier resources when possible

### Resource Cleanup Script

```bash
#!/bin/bash
# cleanup-lab-resources.sh

# Delete S3 buckets
aws s3 rb s3://lab-encrypted-bucket-${ACCOUNT_ID} --force
aws s3 rb s3://cloudtrail-logs-${ACCOUNT_ID} --force

# Delete CloudTrail
aws cloudtrail delete-trail --name lab-trail

# Delete VPC and associated resources
aws ec2 delete-security-group --group-id $WEB_SG_ID
aws ec2 delete-security-group --group-id $DB_SG_ID
aws ec2 detach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET_ID
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET_ID
aws ec2 delete-vpc --vpc-id $VPC_ID

# Delete KMS key (schedule deletion)
aws kms schedule-key-deletion --key-id $KEY_ID --pending-window-in-days 7

# Delete IAM resources
aws iam detach-user-policy --user-name lab-user-readonly --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam delete-user --user-name lab-user-readonly

echo "Cleanup complete"
```

This skill provides comprehensive guidance for completing AWS cloud security labs in the IKB42603 course, covering IAM, encryption, networking, and monitoring with practical, executable examples.
