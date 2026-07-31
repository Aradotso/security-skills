---
name: ikb42603-cloud-security-essentials
description: AWS cloud security laboratory exercises covering IAM, encryption, network security, and monitoring for cloud computing security education
triggers:
  - help me with AWS cloud security labs
  - set up IAM security in AWS
  - configure AWS encryption and key management
  - implement AWS network security with VPC
  - set up CloudWatch and CloudTrail monitoring
  - complete cloud computing security lab
  - configure AWS security groups and access control
  - implement AWS multi-tenancy isolation
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides expertise in completing AWS cloud computing security laboratory exercises. The project is an educational repository covering five core security domains: IAM and account security, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

## Project Overview

IKB42603 is a structured learning path for cloud security fundamentals using AWS services. Each lab builds practical skills in securing cloud infrastructure through hands-on exercises with AWS IAM, VPC, EC2, KMS, CloudTrail, and CloudWatch.

**Core Technologies:**
- AWS IAM (Identity and Access Management)
- AWS VPC (Virtual Private Cloud)
- AWS EC2 (Elastic Compute Cloud)
- AWS KMS (Key Management Service)
- AWS CloudTrail (API logging)
- AWS CloudWatch (Monitoring)
- AWS CLI (Command Line Interface)

## Repository Setup

### Initial Repository Creation

```bash
# Clone the repository template
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create lab structure
mkdir -p screenshots/{lab1,lab2,lab3,lab4,lab5}

# Create lab documentation files
touch Lab0_Environment_Setup.md
touch Lab1_Account_Security_and_IAM.md
touch Lab2_Secure_Isolation_and_Multitenancy.md
touch Lab3_Encryption_and_Key_Management.md
touch Lab4_Access_Control_and_Network_Security.md
touch Lab5_Monitoring_Logging_and_Incident_Detection.md
```

### Repository Structure

```
IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/
├── README.md
├── Lab0_Environment_Setup.md
├── Lab1_Account_Security_and_IAM.md
├── Lab2_Secure_Isolation_and_Multitenancy.md
├── Lab3_Encryption_and_Key_Management.md
├── Lab4_Access_Control_and_Network_Security.md
├── Lab5_Monitoring_Logging_and_Incident_Detection.md
└── screenshots/
    ├── lab1/
    ├── lab2/
    ├── lab3/
    ├── lab4/
    └── lab5/
```

## Lab 1: Account Security and IAM

### Objectives
- Configure AWS root account security
- Create and manage IAM users, groups, and roles
- Implement MFA (Multi-Factor Authentication)
- Apply least privilege access policies

### IAM User Creation

```bash
# Configure AWS CLI with credentials
aws configure
# AWS Access Key ID: [from AWS_ACCESS_KEY_ID env var]
# AWS Secret Access Key: [from AWS_SECRET_ACCESS_KEY env var]
# Default region: us-east-1
# Default output format: json

# Create IAM user
aws iam create-user --user-name lab-user-01

# Create IAM group
aws iam create-group --group-name developers

# Add user to group
aws iam add-user-to-group --user-name lab-user-01 --group-name developers

# List IAM users
aws iam list-users
```

### IAM Policy Creation

```bash
# Create custom IAM policy for read-only S3 access
cat > s3-readonly-policy.json <<EOF
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
        "arn:aws:s3:::my-lab-bucket",
        "arn:aws:s3:::my-lab-bucket/*"
      ]
    }
  ]
}
EOF

# Create the policy
aws iam create-policy \
  --policy-name S3ReadOnlyPolicy \
  --policy-document file://s3-readonly-policy.json

# Attach policy to group
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::ACCOUNT_ID:policy/S3ReadOnlyPolicy
```

### Enable MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-mfa \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA for user (requires two consecutive MFA codes)
aws iam enable-mfa-device \
  --user-name lab-user-01 \
  --serial-number arn:aws:iam::ACCOUNT_ID:mfa/lab-user-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### IAM Role Creation for EC2

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
  --role-name EC2-S3-Access-Role \
  --assume-role-policy-document file://ec2-trust-policy.json

# Attach managed policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Access-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile \
  --role-name EC2-S3-Access-Role
```

## Lab 2: Secure Isolation and Multitenancy

### VPC Creation

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

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id $VPC_ID \
  --enable-dns-hostnames
```

### Subnet Configuration

```bash
# Create public subnet
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'

# Get subnet IDs
PUBLIC_SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=PublicSubnet" \
  --query 'Subnets[0].SubnetId' \
  --output text)

PRIVATE_SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=PrivateSubnet" \
  --query 'Subnets[0].SubnetId' \
  --output text)
```

### Internet Gateway and NAT Gateway

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=LabIGW}]'

IGW_ID=$(aws ec2 describe-internet-gateways \
  --filters "Name=tag:Name,Values=LabIGW" \
  --query 'InternetGateways[0].InternetGatewayId' \
  --output text)

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID

# Allocate Elastic IP for NAT Gateway
aws ec2 allocate-address --domain vpc

EIP_ALLOC_ID=$(aws ec2 describe-addresses \
  --query 'Addresses[0].AllocationId' \
  --output text)

# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
  --subnet-id $PUBLIC_SUBNET_ID \
  --allocation-id $EIP_ALLOC_ID \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=LabNAT}]'

# Wait for NAT Gateway to become available
NAT_GW_ID=$(aws ec2 describe-nat-gateways \
  --filter "Name=tag:Name,Values=LabNAT" \
  --query 'NatGateways[0].NatGatewayId' \
  --output text)
```

### Route Table Configuration

```bash
# Create public route table
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRT}]'

PUBLIC_RT_ID=$(aws ec2 describe-route-tables \
  --filters "Name=tag:Name,Values=PublicRT" \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id $PUBLIC_RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associate public subnet with public route table
aws ec2 associate-route-table \
  --subnet-id $PUBLIC_SUBNET_ID \
  --route-table-id $PUBLIC_RT_ID

# Create private route table
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PrivateRT}]'

PRIVATE_RT_ID=$(aws ec2 describe-route-tables \
  --filters "Name=tag:Name,Values=PrivateRT" \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

# Add route to NAT Gateway
aws ec2 create-route \
  --route-table-id $PRIVATE_RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GW_ID

# Associate private subnet with private route table
aws ec2 associate-route-table \
  --subnet-id $PRIVATE_SUBNET_ID \
  --route-table-id $PRIVATE_RT_ID
```

## Lab 3: Encryption and Key Management

### KMS Key Creation

```bash
# Create customer managed KMS key
aws kms create-key \
  --description "Lab encryption key for S3 and EBS" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Store key ID
KEY_ID=$(aws kms list-keys --query 'Keys[0].KeyId' --output text)

# Create key alias
aws kms create-alias \
  --alias-name alias/lab-encryption-key \
  --target-key-id $KEY_ID

# Describe key
aws kms describe-key --key-id alias/lab-encryption-key
```

### S3 Bucket Encryption

```bash
# Create S3 bucket
aws s3 mb s3://my-encrypted-lab-bucket-$(date +%s)

BUCKET_NAME=my-encrypted-lab-bucket-$(date +%s)

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "aws:kms",
          "KMSMasterKeyID": "alias/lab-encryption-key"
        },
        "BucketKeyEnabled": true
      }
    ]
  }'

# Verify encryption configuration
aws s3api get-bucket-encryption --bucket $BUCKET_NAME

# Upload encrypted file
echo "Sensitive data" > test-file.txt
aws s3 cp test-file.txt s3://$BUCKET_NAME/ \
  --sse aws:kms \
  --sse-kms-key-id alias/lab-encryption-key
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/lab-encryption-key \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EncryptedVolume}]'

# List encrypted volumes
aws ec2 describe-volumes \
  --filters "Name=encrypted,Values=true" \
  --query 'Volumes[*].[VolumeId,Encrypted,KmsKeyId]' \
  --output table
```

### KMS Key Rotation

```bash
# Enable automatic key rotation
aws kms enable-key-rotation --key-id alias/lab-encryption-key

# Check rotation status
aws kms get-key-rotation-status --key-id alias/lab-encryption-key
```

## Lab 4: Access Control and Network Security

### Security Group Configuration

```bash
# Create web server security group
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Security group for web servers" \
  --vpc-id $VPC_ID

WEB_SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=web-server-sg" \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_IP_ADDRESS/32

# Create database security group
aws ec2 create-security-group \
  --group-name database-sg \
  --description "Security group for database servers" \
  --vpc-id $VPC_ID

DB_SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=database-sg" \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

# Allow MySQL only from web server security group
aws ec2 authorize-security-group-ingress \
  --group-id $DB_SG_ID \
  --protocol tcp \
  --port 3306 \
  --source-group $WEB_SG_ID
```

### Network ACL Configuration

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=LabNACL}]'

NACL_ID=$(aws ec2 describe-network-acls \
  --filters "Name=tag:Name,Values=LabNACL" \
  --query 'NetworkAcls[0].NetworkAclId' \
  --output text)

# Allow inbound HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow

# Allow inbound HTTPS
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 110 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow

# Allow ephemeral ports for return traffic
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 120 \
  --protocol tcp \
  --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow

# Allow all outbound traffic
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --egress \
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
# Create CloudWatch Log Group
aws logs create-log-group --log-group-name /aws/vpc/flowlogs

# Create IAM role for VPC Flow Logs
cat > flowlogs-trust-policy.json <<EOF
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
  --assume-role-policy-document file://flowlogs-trust-policy.json

# Attach policy to role
cat > flowlogs-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name VPCFlowLogsRole \
  --policy-name VPCFlowLogsPolicy \
  --policy-document file://flowlogs-policy.json

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids $VPC_ID \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::ACCOUNT_ID:role/VPCFlowLogsRole
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Configuration

```bash
# Create S3 bucket for CloudTrail logs
TRAIL_BUCKET=cloudtrail-logs-$(date +%s)
aws s3 mb s3://$TRAIL_BUCKET

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
      "Resource": "arn:aws:s3:::$TRAIL_BUCKET"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::$TRAIL_BUCKET/AWSLogs/ACCOUNT_ID/*",
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
  --bucket $TRAIL_BUCKET \
  --policy file://cloudtrail-bucket-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name LabTrail \
  --s3-bucket-name $TRAIL_BUCKET \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name LabTrail

# Verify trail status
aws cloudtrail get-trail-status --name LabTrail
```

### CloudWatch Alarms

```bash
# Create SNS topic for alarms
aws sns create-topic --name SecurityAlerts

SNS_TOPIC_ARN=$(aws sns list-topics \
  --query "Topics[?contains(TopicArn, 'SecurityAlerts')].TopicArn" \
  --output text)

# Subscribe email to topic
aws sns subscribe \
  --topic-arn $SNS_TOPIC_ARN \
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
  --alarm-actions $SNS_TOPIC_ARN

# Create metric filter for unauthorized calls
aws logs put-metric-filter \
  --log-group-name CloudTrail/DefaultLogGroup \
  --filter-name UnauthorizedAPICalls \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
    metricName=UnauthorizedAPICalls,metricNamespace=AWS/CloudTrail,metricValue=1
```

### CloudWatch Log Insights Queries

```bash
# Query failed login attempts
aws logs start-query \
  --log-group-name CloudTrail/DefaultLogGroup \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, userIdentity.principalId, errorCode, errorMessage
| filter eventName = "ConsoleLogin" and errorCode = "Failed authentication"
| sort @timestamp desc
| limit 20'

# Query S3 bucket policy changes
aws logs start-query \
  --log-group-name CloudTrail/DefaultLogGroup \
  --start-time $(date -u -d '24 hours ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, userIdentity.principalId, eventName, requestParameters.bucketName
| filter eventName in ["PutBucketPolicy", "DeleteBucketPolicy", "PutBucketAcl"]
| sort @timestamp desc'

# Query IAM policy changes
aws logs start-query \
  --log-group-name CloudTrail/DefaultLogGroup \
  --start-time $(date -u -d '7 days ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, userIdentity.principalId, eventName, requestParameters
| filter eventName like /^(Create|Delete|Put|Attach|Detach).*Policy/
| sort @timestamp desc'
```

### GuardDuty Integration

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

DETECTOR_ID=$(aws guardduty list-detectors \
  --query 'DetectorIds[0]' \
  --output text)

# List findings
aws guardduty list-findings \
  --detector-id $DETECTOR_ID \
  --max-results 50

# Get finding details
FINDING_ID=$(aws guardduty list-findings \
  --detector-id $DETECTOR_ID \
  --query 'FindingIds[0]' \
  --output text)

aws guardduty get-findings \
  --detector-id $DETECTOR_ID \
  --finding-ids $FINDING_ID
```

## Common Patterns

### Lab Documentation Template

```markdown
# Lab X: [Title]

## Objective
[Brief description of what this lab covers]

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Environment
- AWS Region: us-east-1
- Services Used: [list services]
- Prerequisites: [any setup required]

## Implementation Steps

### Step 1: [Task Name]

**Command:**
```bash
[command here]
```

**Expected Output:**
```
[output here]
```

**Screenshot:**
![Step 1](screenshots/labX/step1.png)

### Step 2: [Next Task]

[Continue pattern...]

## Challenges Encountered
- Challenge 1 and how it was resolved
- Challenge 2 and resolution

## Lessons Learned
- Key takeaway 1
- Key takeaway 2

## References
- [AWS Documentation Link]
- [Related Articles]

## Evidence
All screenshots are available in `screenshots/labX/`
```

### Git Workflow for Lab Submissions

```bash
# Start new lab
git checkout -b lab-X

# Make changes and test

# Add all changes
git add .

# Commit with descriptive message
git commit -m "Complete Lab X: [Brief Description]

- Configured [feature 1]
- Implemented [feature 2]
- Added screenshots and documentation"

# Push to remote
git push origin lab-X

# Merge to main after completion
git checkout main
git merge lab-X
git push origin main
```

## Troubleshooting

### AWS CLI Configuration Issues

```bash
# Verify AWS CLI installation
aws --version

# Check current configuration
aws configure list

# Test credentials
aws sts get-caller-identity

# Set region explicitly
export AWS_DEFAULT_REGION=us-east-1

# Use named profiles
aws configure --profile lab-profile
aws s3 ls --profile lab-profile
```

### IAM Permission Errors

```bash
# Check effective permissions for current user
aws iam get-user

aws iam list-attached-user-policies --user-name $(aws iam get-user --query 'User.UserName' --output text)

# Check group memberships
aws iam list-groups-for-user --user-name $(aws iam get-user --query 'User.UserName' --output text)

# Simulate policy
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT_ID:user/lab-user-01 \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-lab-bucket/*
```

### VPC Connectivity Issues

```bash
# Verify VPC configuration
aws ec2 describe-vpcs --vpc-ids $VPC_ID

# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID"

# Check security group rules
aws ec2 describe-security-groups --group-ids $WEB_SG_ID

# Check NACL rules
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=$VPC_ID"

# Test connectivity from EC2 instance
aws ssm start-session --target i-1234567890abcdef0
# Once connected:
curl -I http://example.com
ping 8.8.8.8
```

### CloudTrail Not Logging

```bash
# Verify trail is active
aws cloudtrail describe-trails --trail-name-list LabTrail

# Check trail status
aws cloudtrail get-trail-status --name LabTrail

# Verify S3 bucket policy
aws s3api get-bucket-policy --bucket $TRAIL_BUCKET

# Check for recent events
aws cloudtrail lookup-events \
  --max-results 10 \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
```

### KMS Key Access Issues

```bash
# List KMS keys
aws kms list-keys

# Check key policy
aws kms get-key-policy \
  --key-id alias/lab-encryption-key \
  --policy-name default

# Verify key grants
aws kms list-grants --key-id alias/lab-encryption-key

# Test encryption
echo "test data" > test.txt
aws kms encrypt \
  --key-id alias/lab-encryption-key \
  --plaintext fileb://test.txt \
  --query CiphertextBlob \
  --output text | base64 --decode > encrypted.bin
```

## Best Practices

1. **Always use environment variables for credentials** - Never hardcode AWS credentials
2. **Tag all resources** - Use consistent naming conventions for easy identification
3. **Enable MFA** - Protect root and IAM user accounts with multi-factor authentication
4. **Implement least privilege** - Grant only the minimum
