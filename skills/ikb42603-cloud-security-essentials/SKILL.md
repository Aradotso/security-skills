---
name: ikb42603-cloud-security-essentials
description: AWS cloud security lab exercises covering IAM, encryption, VPC isolation, access control, and monitoring for educational purposes
triggers:
  - "help me with AWS cloud security labs"
  - "show me how to set up IAM users and policies"
  - "configure AWS encryption and KMS"
  - "set up VPC isolation and security groups"
  - "implement CloudWatch and CloudTrail monitoring"
  - "complete cloud computing security exercises"
  - "document AWS security lab work"
  - "troubleshoot AWS security configurations"
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This repository provides a structured curriculum for learning AWS cloud security fundamentals through hands-on laboratory exercises. It covers five core security domains: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

The labs use AWS services including IAM, VPC, EC2, Security Groups, KMS, CloudTrail, and CloudWatch. Each lab is documented separately with implementation steps, screenshots, and learning outcomes.

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

## Getting Started

### Initial Repository Setup

```bash
# Clone the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create lab documentation files
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

### AWS CLI Configuration

```bash
# Configure AWS CLI with your credentials
aws configure
# AWS Access Key ID: [use environment variable AWS_ACCESS_KEY_ID]
# AWS Secret Access Key: [use environment variable AWS_SECRET_ACCESS_KEY]
# Default region name: us-east-1
# Default output format: json

# Verify configuration
aws sts get-caller-identity
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create an IAM user
aws iam create-user --user-name student-user

# Create access keys for the user
aws iam create-access-key --user-name student-user

# Attach a policy to the user
aws iam attach-user-policy \
  --user-name student-user \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Creating IAM Groups and Roles

```bash
# Create an IAM group
aws iam create-group --group-name developers

# Add user to group
aws iam add-user-to-group \
  --user-name student-user \
  --group-name developers

# Create a custom policy
cat > s3-read-policy.json <<EOF
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
        "arn:aws:s3:::my-bucket/*",
        "arn:aws:s3:::my-bucket"
      ]
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name S3ReadOnlyCustom \
  --policy-document file://s3-read-policy.json
```

### Enabling MFA

```bash
# Enable MFA for root account (done via Console)
# Create virtual MFA device for IAM user
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name student-mfa \
  --outfile qr-code.png \
  --bootstrap-method QRCodePNG

# Enable MFA device (after scanning QR code)
aws iam enable-mfa-device \
  --user-name student-user \
  --serial-number arn:aws:iam::ACCOUNT_ID:mfa/student-mfa \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

## Lab 2: Secure Isolation and Multitenancy

### Creating VPCs with Isolation

```bash
# Create VPC for tenant 1
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Tenant1-VPC}]'

# Create VPC for tenant 2
aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Tenant2-VPC}]'

# Create subnet in Tenant1 VPC
aws ec2 create-subnet \
  --vpc-id vpc-0123456789abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Tenant1-Subnet}]'

# Create internet gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Tenant1-IGW}]'

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway \
  --vpc-id vpc-0123456789abcdef0 \
  --internet-gateway-id igw-0123456789abcdef0
```

### Configuring Route Tables

```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-0123456789abcdef0 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Tenant1-RT}]'

# Add route to internet gateway
aws ec2 create-route \
  --route-table-id rtb-0123456789abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0123456789abcdef0

# Associate route table with subnet
aws ec2 associate-route-table \
  --subnet-id subnet-0123456789abcdef0 \
  --route-table-id rtb-0123456789abcdef0
```

## Lab 3: Encryption and Key Management

### Creating KMS Keys

```bash
# Create customer managed key
aws kms create-key \
  --description "Lab encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create an alias for the key
aws kms create-alias \
  --alias-name alias/lab-key \
  --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab

# List KMS keys
aws kms list-keys

# Describe key
aws kms describe-key --key-id alias/lab-key
```

### Encrypting S3 Buckets

```bash
# Create S3 bucket
aws s3 mb s3://my-encrypted-bucket-$RANDOM

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket my-encrypted-bucket-12345 \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab-key"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Upload encrypted file
aws s3 cp file.txt s3://my-encrypted-bucket-12345/ \
  --sse aws:kms \
  --sse-kms-key-id alias/lab-key
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/lab-key \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EncryptedVolume}]'

# Enable encryption by default for account
aws ec2 enable-ebs-encryption-by-default
```

## Lab 4: Access Control and Network Security

### Creating Security Groups

```bash
# Create security group for web servers
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Security group for web servers" \
  --vpc-id vpc-0123456789abcdef0

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/32
```

### Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-0123456789abcdef0 \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Custom-NACL}]'

# Add inbound rule to allow HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0123456789abcdef0 \
  --ingress \
  --rule-number 100 \
  --protocol 6 \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Add outbound rule
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0123456789abcdef0 \
  --egress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow
```

### Launching EC2 with Security Groups

```bash
# Launch EC2 instance with security group
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]'
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Enabling CloudTrail

```bash
# Create S3 bucket for CloudTrail logs
aws s3 mb s3://cloudtrail-logs-$ACCOUNT_ID-$RANDOM

# Apply bucket policy for CloudTrail
cat > bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::cloudtrail-logs-$ACCOUNT_ID-12345"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::cloudtrail-logs-$ACCOUNT_ID-12345/*",
      "Condition": {
        "StringEquals": {"s3:x-amz-acl": "bucket-owner-full-control"}
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket cloudtrail-logs-$ACCOUNT_ID-12345 \
  --policy file://bucket-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name cloudtrail-logs-$ACCOUNT_ID-12345

# Start logging
aws cloudtrail start-logging --name my-trail

# Enable log file validation
aws cloudtrail update-trail \
  --name my-trail \
  --enable-log-file-validation
```

### CloudWatch Alarms

```bash
# Create CloudWatch alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICallsEventCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1

# Create alarm for root account usage
aws cloudwatch put-metric-alarm \
  --alarm-name RootAccountUsage \
  --alarm-description "Alert when root account is used" \
  --metric-name RootAccountUsageEventCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1
```

### CloudWatch Log Groups and Filters

```bash
# Create log group for CloudTrail
aws logs create-log-group --log-group-name CloudTrail/DefaultLogGroup

# Create metric filter for unauthorized API calls
aws logs put-metric-filter \
  --log-group-name CloudTrail/DefaultLogGroup \
  --filter-name UnauthorizedAPICalls \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
    metricName=UnauthorizedAPICallsEventCount,metricNamespace=CloudTrailMetrics,metricValue=1

# Query CloudTrail logs
aws logs filter-log-events \
  --log-group-name CloudTrail/DefaultLogGroup \
  --filter-pattern "{ $.eventName = ConsoleLogin }" \
  --start-time $(date -u -d '1 hour ago' +%s)000
```

### AWS Config for Compliance

```bash
# Create configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::$ACCOUNT_ID:role/config-role

# Create delivery channel
aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=config-bucket-$ACCOUNT_ID

# Start configuration recorder
aws configservice start-configuration-recorder \
  --configuration-recorder-name default

# Enable specific config rules
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "encrypted-volumes",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "ENCRYPTED_VOLUMES"
    }
  }'
```

## Lab Documentation Template

Each lab should follow this structure:

```markdown
# Lab X: [Title]

## Objective
Brief description of what the lab aims to achieve.

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Prerequisites
- AWS account with appropriate permissions
- AWS CLI configured
- Basic understanding of [relevant concepts]

## Environment Setup
- Region: us-east-1
- Services used: [list]

## Implementation Steps

### Step 1: [Task Name]
Description of the task.

**Commands:**
```bash
# Command with explanation
aws service command --options
```

**Screenshot:**
![Step 1 Screenshot](./screenshots/lab1-step1.png)

### Step 2: [Task Name]
...

## Verification
How to verify the lab was completed successfully.

```bash
# Verification commands
aws service describe-resource
```

## Challenges Encountered
- Challenge 1: Description and resolution
- Challenge 2: Description and resolution

## Lessons Learned
Key takeaways from completing this lab.

## Cleanup
Commands to remove resources created during the lab.

```bash
# Cleanup commands
aws service delete-resource
```

## References
- [AWS Documentation Link]
- [Additional Resource]
```

## Common Patterns

### IAM Policy Testing

```bash
# Simulate policy evaluation
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::$ACCOUNT_ID:user/student-user \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt

# Test permissions
aws iam get-user
aws s3 ls s3://my-bucket/
```

### Resource Tagging for Organization

```bash
# Tag resources consistently
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags Key=Lab,Value=Lab1 Key=Environment,Value=Learning

# List resources by tag
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Lab,Values=Lab1
```

### Cost Management

```bash
# Enable AWS Cost Explorer
# Create budget alert
aws budgets create-budget \
  --account-id $ACCOUNT_ID \
  --budget '{
    "BudgetName": "Lab-Budget",
    "BudgetLimit": {"Amount": "10", "Unit": "USD"},
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }'
```

## Troubleshooting

### Permission Issues

```bash
# Check current identity
aws sts get-caller-identity

# Check user permissions
aws iam list-attached-user-policies --user-name student-user
aws iam list-user-policies --user-name student-user

# Check group memberships
aws iam list-groups-for-user --user-name student-user
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-0123456789abcdef0"

# Check security group rules
aws ec2 describe-security-groups --group-ids sg-0123456789abcdef0

# Check network ACLs
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=vpc-0123456789abcdef0"

# Test connectivity from EC2
aws ec2-instance-connect ssh --instance-id i-1234567890abcdef0
```

### CloudTrail Not Logging

```bash
# Check trail status
aws cloudtrail get-trail-status --name my-trail

# Verify S3 bucket permissions
aws s3api get-bucket-policy --bucket cloudtrail-logs-$ACCOUNT_ID-12345

# Check for recent events
aws cloudtrail lookup-events --max-results 10
```

### KMS Key Access Issues

```bash
# List key policies
aws kms get-key-policy \
  --key-id alias/lab-key \
  --policy-name default

# Check key grants
aws kms list-grants --key-id alias/lab-key

# Test encryption
echo "test data" > test.txt
aws kms encrypt \
  --key-id alias/lab-key \
  --plaintext fileb://test.txt \
  --output text \
  --query CiphertextBlob
```

## Git Workflow for Labs

```bash
# Create branch for each lab
git checkout -b lab1-iam-security

# Make changes and commit frequently
git add Lab1_Account_Security_and_IAM.md screenshots/
git commit -m "Complete Lab 1: Configure IAM users and MFA"

# Push to remote
git push origin lab1-iam-security

# Merge to main after completion
git checkout main
git merge lab1-iam-security
git push origin main

# Tag completed labs
git tag -a lab1-complete -m "Lab 1 completed and verified"
git push origin lab1-complete
```

## Best Practices

1. **Always use environment variables for credentials**
   ```bash
   export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
   export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
   export AWS_DEFAULT_REGION=us-east-1
   ```

2. **Document every configuration change with screenshots**

3. **Use consistent naming conventions**
   - Resources: `lab1-resource-name`
   - Tags: `Lab=Lab1`, `Student=[Name]`

4. **Clean up resources after each lab to avoid charges**

5. **Enable MFA on all accounts**

6. **Follow principle of least privilege for IAM**

7. **Enable logging and monitoring from the start**

8. **Commit code regularly with descriptive messages**

9. **Test configurations before finalizing documentation**

10. **Review security group rules for overly permissive access**
