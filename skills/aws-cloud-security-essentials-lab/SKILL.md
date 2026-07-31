---
name: aws-cloud-security-essentials-lab
description: Educational lab repository for AWS cloud computing security fundamentals including IAM, VPC, encryption, monitoring, and incident detection
triggers:
  - how do I set up AWS IAM security labs
  - show me cloud security lab exercises
  - help with AWS security essentials coursework
  - configure AWS encryption and key management lab
  - set up AWS monitoring and logging lab
  - create secure multi-tenant cloud environment
  - implement AWS access control and network security
  - troubleshoot AWS CloudTrail and CloudWatch setup
---

# AWS Cloud Security Essentials Lab

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides guidance for working with the IKB42603 Cloud Computing Security Essentials lab repository, which contains hands-on exercises covering fundamental AWS security concepts including IAM, VPC isolation, encryption, access control, and security monitoring.

## What This Project Does

This is an educational repository structured around five core AWS security labs:

- **Lab 1**: Account Security and IAM (Identity and Access Management)
- **Lab 2**: Secure Isolation and Multitenancy (VPC, Security Groups)
- **Lab 3**: Encryption and Key Management (KMS, data protection)
- **Lab 4**: Access Control and Network Security (Network ACLs, Security Groups)
- **Lab 5**: Monitoring, Logging, and Incident Detection (CloudTrail, CloudWatch)

Each lab teaches practical AWS security implementation through hands-on exercises.

## Repository Setup

### Initial Repository Creation

```bash
# Clone the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create lab structure
mkdir -p Lab{1..5}
touch Lab0_Environment_Setup.md
touch Lab1_Account_Security_and_IAM.md
touch Lab2_Secure_Isolation_and_Multitenancy.md
touch Lab3_Encryption_and_Key_Management.md
touch Lab4_Access_Control_and_Network_Security.md
touch Lab5_Monitoring_Logging_and_Incident_Detection.md

# Initial commit
git add .
git commit -m "Initial lab structure setup"
git push origin main
```

### Standard Lab Documentation Structure

Each lab markdown file should follow this structure:

```markdown
# Lab X - [Lab Title]

## Objective
[What you will accomplish]

## Learning Outcomes
- Outcome 1
- Outcome 2

## Prerequisites
- AWS Account
- AWS CLI installed
- Appropriate IAM permissions

## Environment Setup
[Required tools and configuration]

## Implementation Steps

### Step 1: [Task Name]
[Description]

**Commands:**
```bash
# Command with explanation
aws [service] [action] --options
```

**Screenshot:**
![Description](./screenshots/step1.png)

### Step 2: [Next Task]
...

## Verification
[How to verify successful completion]

## Challenges Encountered
[Document any issues and solutions]

## Lessons Learned
[Key takeaways]

## Cleanup
```bash
# Commands to remove resources
```

## References
- [Documentation links]
```

## Lab 1: Account Security and IAM

### Creating IAM Users with MFA

```bash
# Create IAM user
aws iam create-user --user-name lab-user-01

# Create login profile
aws iam create-login-profile \
  --user-name lab-user-01 \
  --password 'TempPassword123!' \
  --password-reset-required

# Attach policy for MFA enforcement
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::aws:policy/IAMUserChangePassword

# Enable virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-01-mfa \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG
```

### Creating Custom IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::lab-bucket-*",
        "arn:aws:s3:::lab-bucket-*/*"
      ]
    },
    {
      "Sid": "DenyWithoutMFA",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

```bash
# Create the policy
aws iam create-policy \
  --policy-name LabS3ReadOnlyWithMFA \
  --policy-document file://policy.json

# Attach to user
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::ACCOUNT_ID:policy/LabS3ReadOnlyWithMFA
```

## Lab 2: Secure Isolation and Multitenancy

### Creating Isolated VPC

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-secure-vpc}]'

# Store VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=lab-secure-vpc" \
  --query 'Vpcs[0].VpcId' \
  --output text)

# Create public subnet
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-public-subnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab-private-subnet}]'

# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab-igw}]'

IGW_ID=$(aws ec2 describe-internet-gateways \
  --filters "Name=tag:Name,Values=lab-igw" \
  --query 'InternetGateways[0].InternetGatewayId' \
  --output text)

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
```

### Security Groups for Multi-tier Architecture

```bash
# Create web tier security group
aws ec2 create-security-group \
  --group-name lab-web-sg \
  --description "Security group for web tier" \
  --vpc-id $VPC_ID

WEB_SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=lab-web-sg" \
  --query 'SecurityGroups[0].GroupId' \
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

# Create database tier security group
aws ec2 create-security-group \
  --group-name lab-db-sg \
  --description "Security group for database tier" \
  --vpc-id $VPC_ID

DB_SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=lab-db-sg" \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

# Allow MySQL only from web tier
aws ec2 authorize-security-group-ingress \
  --group-id $DB_SG_ID \
  --protocol tcp \
  --port 3306 \
  --source-group $WEB_SG_ID
```

## Lab 3: Encryption and Key Management

### Creating and Using KMS Keys

```bash
# Create customer managed key
aws kms create-key \
  --description "Lab encryption key for S3" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

KEY_ID=$(aws kms list-keys --query 'Keys[0].KeyId' --output text)

# Create alias
aws kms create-alias \
  --alias-name alias/lab-s3-key \
  --target-key-id $KEY_ID

# Create encrypted S3 bucket
aws s3api create-bucket \
  --bucket lab-encrypted-bucket-$(date +%s) \
  --region us-east-1

BUCKET_NAME=$(aws s3api list-buckets \
  --query 'Buckets[?contains(Name, `lab-encrypted-bucket`)].Name' \
  --output text)

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "'$KEY_ID'"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Upload encrypted file
echo "Sensitive data" > test-file.txt
aws s3 cp test-file.txt s3://$BUCKET_NAME/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id $KEY_ID
```

### Encrypting EBS Volumes

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id $KEY_ID \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=lab-encrypted-volume}]'

# Verify encryption
aws ec2 describe-volumes \
  --filters "Name=tag:Name,Values=lab-encrypted-volume" \
  --query 'Volumes[0].[Encrypted,KmsKeyId]' \
  --output table
```

## Lab 4: Access Control and Network Security

### Network ACL Configuration

```bash
# Get subnet ID
SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=lab-public-subnet" \
  --query 'Subnets[0].SubnetId' \
  --output text)

# Create Network ACL
aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=lab-nacl}]'

NACL_ID=$(aws ec2 describe-network-acls \
  --filters "Name=tag:Name,Values=lab-nacl" \
  --query 'NetworkAcls[0].NetworkAclId' \
  --output text)

# Allow inbound HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow \
  --ingress

# Allow inbound HTTPS
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 110 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow \
  --ingress

# Deny specific IP range
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 50 \
  --protocol -1 \
  --cidr-block 10.0.100.0/24 \
  --rule-action deny \
  --ingress

# Allow outbound traffic
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow \
  --egress
```

### VPC Flow Logs

```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name /aws/vpc/flowlogs

# Create IAM role for flow logs
cat > trust-policy.json <<EOF
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
  --assume-role-policy-document file://trust-policy.json

# Attach policy
cat > flow-logs-policy.json <<EOF
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
  --policy-document file://flow-logs-policy.json

# Enable flow logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids $VPC_ID \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::ACCOUNT_ID:role/VPCFlowLogsRole
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket lab-cloudtrail-logs-$(date +%s) \
  --region us-east-1

TRAIL_BUCKET=$(aws s3api list-buckets \
  --query 'Buckets[?contains(Name, `lab-cloudtrail-logs`)].Name' \
  --output text)

# Apply bucket policy
cat > bucket-policy.json <<EOF
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
  --policy file://bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name lab-security-trail \
  --s3-bucket-name $TRAIL_BUCKET \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name lab-security-trail

# Enable log file validation
aws cloudtrail update-trail \
  --name lab-security-trail \
  --enable-log-file-validation
```

### CloudWatch Alarms for Security Events

```bash
# Create SNS topic for alerts
aws sns create-topic --name lab-security-alerts

TOPIC_ARN=$(aws sns list-topics \
  --query 'Topics[?contains(TopicArn, `lab-security-alerts`)].TopicArn' \
  --output text)

# Subscribe email
aws sns subscribe \
  --topic-arn $TOPIC_ARN \
  --protocol email \
  --notification-endpoint ${ADMIN_EMAIL}

# Create metric filter for root account usage
aws logs put-metric-filter \
  --log-group-name CloudTrail/logs \
  --filter-name RootAccountUsage \
  --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
  --metric-transformations \
    metricName=RootAccountUsageCount,metricNamespace=CloudTrailMetrics,metricValue=1

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name RootAccountUsageAlarm \
  --alarm-description "Alarm when root account is used" \
  --metric-name RootAccountUsageCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions $TOPIC_ARN

# Metric filter for unauthorized API calls
aws logs put-metric-filter \
  --log-group-name CloudTrail/logs \
  --filter-name UnauthorizedAPICalls \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
    metricName=UnauthorizedAPICallsCount,metricNamespace=CloudTrailMetrics,metricValue=1

aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICallsAlarm \
  --alarm-description "Alarm for unauthorized API calls" \
  --metric-name UnauthorizedAPICallsCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions $TOPIC_ARN
```

### GuardDuty Setup

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)

# Create SNS topic for findings
aws sns create-topic --name lab-guardduty-findings

GUARDDUTY_TOPIC_ARN=$(aws sns list-topics \
  --query 'Topics[?contains(TopicArn, `lab-guardduty-findings`)].TopicArn' \
  --output text)

# Subscribe to findings
aws sns subscribe \
  --topic-arn $GUARDDUTY_TOPIC_ARN \
  --protocol email \
  --notification-endpoint ${SECURITY_EMAIL}

# Create EventBridge rule for GuardDuty findings
cat > guardduty-rule.json <<EOF
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [7, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8, 7.9, 8, 8.0, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9]
  }
}
EOF

aws events put-rule \
  --name lab-guardduty-high-severity \
  --event-pattern file://guardduty-rule.json \
  --state ENABLED

aws events put-targets \
  --rule lab-guardduty-high-severity \
  --targets "Id"="1","Arn"="$GUARDDUTY_TOPIC_ARN"
```

## Common Patterns

### Resource Tagging for Security Compliance

```bash
# Apply consistent tags to all resources
aws ec2 create-tags \
  --resources $VPC_ID $SUBNET_ID $SG_ID \
  --tags \
    Key=Environment,Value=Lab \
    Key=Owner,Value=Student \
    Key=Course,Value=IKB42603 \
    Key=CostCenter,Value=Education \
    Key=DataClassification,Value=Public
```

### Automated Cleanup Script

```bash
#!/bin/bash
# cleanup-lab.sh - Remove all lab resources

# Variables
VPC_NAME="lab-secure-vpc"

# Get VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=$VPC_NAME" \
  --query 'Vpcs[0].VpcId' \
  --output text)

if [ "$VPC_ID" != "None" ]; then
  # Delete NAT Gateways
  for NAT_ID in $(aws ec2 describe-nat-gateways \
    --filter "Name=vpc-id,Values=$VPC_ID" \
    --query 'NatGateways[*].NatGatewayId' \
    --output text); do
    aws ec2 delete-nat-gateway --nat-gateway-id $NAT_ID
    echo "Deleted NAT Gateway: $NAT_ID"
  done

  # Delete EC2 instances
  for INSTANCE_ID in $(aws ec2 describe-instances \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=instance-state-name,Values=running,stopped" \
    --query 'Reservations[*].Instances[*].InstanceId' \
    --output text); do
    aws ec2 terminate-instances --instance-ids $INSTANCE_ID
    echo "Terminated instance: $INSTANCE_ID"
  done

  # Wait for instances to terminate
  aws ec2 wait instance-terminated \
    --filters "Name=vpc-id,Values=$VPC_ID"

  # Delete security groups (except default)
  for SG_ID in $(aws ec2 describe-security-groups \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'SecurityGroups[?GroupName!=`default`].GroupId' \
    --output text); do
    aws ec2 delete-security-group --group-id $SG_ID
    echo "Deleted security group: $SG_ID"
  done

  # Detach and delete internet gateway
  for IGW_ID in $(aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID" \
    --query 'InternetGateways[*].InternetGatewayId' \
    --output text); do
    aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
    aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
    echo "Deleted IGW: $IGW_ID"
  done

  # Delete subnets
  for SUBNET_ID in $(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[*].SubnetId' \
    --output text); do
    aws ec2 delete-subnet --subnet-id $SUBNET_ID
    echo "Deleted subnet: $SUBNET_ID"
  done

  # Delete VPC
  aws ec2 delete-vpc --vpc-id $VPC_ID
  echo "Deleted VPC: $VPC_ID"
fi

echo "Cleanup complete"
```

## Troubleshooting

### AWS CLI Authentication Issues

```bash
# Verify credentials
aws sts get-caller-identity

# Check configuration
aws configure list

# Use named profile
aws configure --profile lab-profile
aws s3 ls --profile lab-profile

# Set environment variables
export AWS_PROFILE=lab-profile
export AWS_DEFAULT_REGION=us-east-1
```

### Permission Denied Errors

```bash
# Check current user permissions
aws iam get-user

# List attached policies
aws iam list-attached-user-policies --user-name $(aws iam get-user --query 'User.UserName' --output text)

# Simulate policy evaluation
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT_ID:user/lab-user \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::bucket-name/*
```

### VPC Resource Deletion Blocked

```bash
# Find dependencies preventing VPC deletion
aws ec2 describe-vpcs --vpc-ids $VPC_ID

# Check for running instances
aws ec2 describe-instances \
  --filters "Name=vpc-id,Values=$VPC_ID" "Name=instance-state-name,Values=running"

# Check for network interfaces
aws ec2 describe-network-interfaces \
  --filters "Name=vpc-id,Values=$VPC_ID"

# Force detach and delete ENIs
for ENI_ID in $(aws ec2 describe-network-interfaces \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'NetworkInterfaces[*].NetworkInterfaceId' \
  --output text); do
  ATTACHMENT_ID=$(aws ec2 describe-network-interfaces \
    --network-interface-ids $ENI_ID \
    --query 'NetworkInterfaces[0].Attachment.AttachmentId' \
    --output text)
  if [ "$ATTACHMENT_ID" != "None" ]; then
    aws ec2 detach-network-interface --attachment-id $ATTACHMENT_ID --force
  fi
  aws ec2 delete-network-interface --network-interface-id $ENI_ID
done
```

### CloudTrail Not Logging

```bash
# Verify trail status
aws cloudtrail get-trail-status --name lab-security-trail

# Check trail configuration
aws cloudtrail describe-trails --trail-name-list lab-security-trail

# Validate S3 bucket permissions
aws s3api get-bucket-policy --bucket $TRAIL_BUCKET

# Test logging
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser \
  --max-results 10
```

### KMS Key Access Issues

```bash
# Check key policy
aws kms get-key-policy \
  --key-id $KEY_ID \
  --policy-name default

# List grants
aws kms list-grants --key-id $KEY_ID

# Verify key state
aws kms describe-key --key-id $KEY_ID --query 'KeyMetadata.KeyState'

# Test encryption
aws kms encrypt \
  --key-id $KEY_ID \
  --plaintext "test" \
  --output text \
  --query CiphertextBlob
```

## Git Workflow for Labs

```bash
# Start new lab
git checkout -b lab-3-encryption
mkdir -p Lab3/screenshots

# Make changes and commit incrementally
git add Lab3/
git commit -m "Lab 3: Create KMS key and configure encryption"

# Continue work
git add Lab3/
git commit -m "Lab 3: Implement S3 bucket encryption"

# Final commit
git add Lab3/
git commit -m "Lab 3: Complete encryption lab with documentation"

# Merge to main
git checkout main
git merge lab-3-encryption
git push origin main
```

## Environment Variables

Always use environment variables for sensitive information:

```bash
# Set in ~/.aws/config or export
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export AWS_DEFAULT_REGION=us-east-1
export ADMIN_EMAIL=admin@example.com
export SECURITY_EMAIL=security@example.com
```

## Additional Resources

- AWS Security Best Practices: https://aws.amazon.com/security/best-practices/
- AWS Well-Architected Framework - Security Pillar
- AWS CLI Command Reference: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/
- CloudTrail Log File Examples
- GuardDuty Finding Types Documentation
