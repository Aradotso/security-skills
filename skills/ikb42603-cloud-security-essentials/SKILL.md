---
name: ikb42603-cloud-security-essentials
description: AWS cloud security lab exercises covering IAM, encryption, VPC isolation, access control, and monitoring for security education
triggers:
  - how do I set up AWS IAM users and policies
  - show me AWS KMS encryption examples
  - configure VPC security groups and network isolation
  - set up AWS CloudTrail and CloudWatch monitoring
  - implement AWS security best practices
  - create secure multi-tenant AWS architecture
  - configure AWS access control and network security
  - AWS security lab exercises and examples
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This project provides hands-on AWS cloud security lab exercises for the IKB42603 Cloud Computing Security Essentials course. It covers five core security domains: Account Security & IAM, Secure Isolation & Multitenancy, Encryption & Key Management, Access Control & Network Security, and Monitoring & Incident Detection.

The labs use AWS services including IAM, VPC, EC2, Security Groups, KMS, CloudTrail, and CloudWatch to teach practical cloud security implementation.

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

## Prerequisites

- AWS Account (Free Tier eligible)
- AWS CLI installed and configured
- Basic understanding of cloud computing concepts
- Git for version control

## Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS
```

### 2. Configure AWS CLI

```bash
aws configure
# Enter:
# AWS Access Key ID: $AWS_ACCESS_KEY_ID
# AWS Secret Access Key: $AWS_SECRET_ACCESS_KEY
# Default region: us-east-1
# Default output format: json
```

### 3. Verify AWS Connection

```bash
aws sts get-caller-identity
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-01

# Create access keys for programmatic access
aws iam create-access-key --user-name lab-user-01

# Attach a managed policy
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
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
        "arn:aws:s3:::lab-bucket-*",
        "arn:aws:s3:::lab-bucket-*/*"
      ]
    }
  ]
}
```

```bash
# Create the policy
aws iam create-policy \
  --policy-name S3ReadOnlyLabPolicy \
  --policy-document file://s3-readonly-policy.json

# Attach to user
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/S3ReadOnlyLabPolicy
```

### Enabling MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-mfa-device \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA (after scanning QR code)
aws iam enable-mfa-device \
  --user-name lab-user-01 \
  --serial-number arn:aws:iam::$AWS_ACCOUNT_ID:mfa/lab-mfa-device \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### Creating IAM Groups

```bash
# Create group
aws iam create-group --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-01 \
  --group-name Developers
```

## Lab 2: Secure Isolation and Multitenancy

### Creating VPC with Isolation

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab-VPC}]'

# Create public subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-Subnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Private-Subnet}]'
```

### Security Groups for Multi-Tenant Isolation

```bash
# Create security group for tenant A
aws ec2 create-security-group \
  --group-name tenant-a-sg \
  --description "Security group for Tenant A" \
  --vpc-id vpc-xxxxx

# Allow inbound HTTP only from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 203.0.113.0/24

# Create security group for tenant B (isolated)
aws ec2 create-security-group \
  --group-name tenant-b-sg \
  --description "Security group for Tenant B" \
  --vpc-id vpc-xxxxx
```

### Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-xxxxx \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Private-NACL}]'

# Add rule to deny traffic from specific subnet
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --ingress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 10.0.1.0/24 \
  --rule-action deny
```

## Lab 3: Encryption and Key Management

### Creating KMS Keys

```bash
# Create customer managed key
aws kms create-key \
  --description "Lab encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create alias
aws kms create-alias \
  --alias-name alias/lab-key \
  --target-key-id <key-id>
```

### Encrypting Data with KMS

```bash
# Encrypt plaintext
echo "Sensitive data" > plaintext.txt
aws kms encrypt \
  --key-id alias/lab-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode
```

### S3 Bucket Encryption

```bash
# Create bucket with encryption
aws s3api create-bucket \
  --bucket lab-encrypted-bucket-$RANDOM \
  --region us-east-1

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket lab-encrypted-bucket-$RANDOM \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab-key"
      }
    }]
  }'
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --size 10 \
  --availability-zone us-east-1a \
  --encrypted \
  --kms-key-id alias/lab-key \
  --volume-type gp3

# Encrypt existing volume snapshot
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-xxxxx \
  --encrypted \
  --kms-key-id alias/lab-key
```

## Lab 4: Access Control and Network Security

### Configuring Security Groups

```bash
# Create web server security group
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Allow HTTP/HTTPS" \
  --vpc-id vpc-xxxxx

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 22 \
  --cidr $MY_IP/32
```

### VPC Endpoints for Private Access

```bash
# Create S3 VPC endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxxxx

# Create interface endpoint for EC2
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-east-1.ec2 \
  --subnet-ids subnet-xxxxx \
  --security-group-ids sg-xxxxx
```

### IAM Roles for EC2

```json
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
```

```bash
# Create role
aws iam create-role \
  --role-name EC2-S3-Access-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach policy
aws iam attach-role-policy \
  --role-name EC2-S3-Access-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Profile

# Add role to profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Profile \
  --role-name EC2-S3-Access-Role
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Enabling CloudTrail

```bash
# Create S3 bucket for logs
aws s3api create-bucket \
  --bucket lab-cloudtrail-logs-$AWS_ACCOUNT_ID \
  --region us-east-1

# Create trail
aws cloudtrail create-trail \
  --name lab-security-trail \
  --s3-bucket-name lab-cloudtrail-logs-$AWS_ACCOUNT_ID

# Start logging
aws cloudtrail start-logging \
  --name lab-security-trail

# Verify trail status
aws cloudtrail get-trail-status \
  --name lab-security-trail
```

### CloudWatch Alarms for Security Events

```bash
# Create SNS topic for alerts
aws sns create-topic --name SecurityAlerts

# Subscribe email
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:$AWS_ACCOUNT_ID:SecurityAlerts \
  --protocol email \
  --notification-endpoint $EMAIL_ADDRESS

# Create metric filter for unauthorized API calls
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/logs \
  --filter-name UnauthorizedAPICalls \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
    metricName=UnauthorizedAPICallsCount,metricNamespace=CloudTrailMetrics,metricValue=1

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICallsAlarm \
  --alarm-description "Triggers on unauthorized API calls" \
  --metric-name UnauthorizedAPICallsCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:us-east-1:$AWS_ACCOUNT_ID:SecurityAlerts
```

### VPC Flow Logs

```bash
# Create log group
aws logs create-log-group \
  --log-group-name /aws/vpc/flowlogs

# Create IAM role for Flow Logs
aws iam create-role \
  --role-name VPCFlowLogsRole \
  --assume-role-policy-document file://vpc-flow-logs-trust-policy.json

# Create flow log
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::$AWS_ACCOUNT_ID:role/VPCFlowLogsRole
```

### Query CloudTrail Logs

```bash
# List recent events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser \
  --max-results 10

# Search for specific user activity
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=lab-user-01 \
  --start-time 2026-07-27T00:00:00Z \
  --end-time 2026-07-28T00:00:00Z
```

## Common Patterns

### Least Privilege IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSpecificS3Actions",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```

### Defense in Depth VPC Architecture

```bash
# Create layered security: NACL + Security Group + WAF
# Layer 1: NACL (network level)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Layer 2: Security Group (instance level)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 443 \
  --source-group sg-alb-xxxxx

# Layer 3: Application-level logging
```

### Automated Security Scanning

```bash
#!/bin/bash
# Check for public S3 buckets
aws s3api list-buckets --query "Buckets[].Name" --output text | \
while read bucket; do
  acl=$(aws s3api get-bucket-acl --bucket $bucket 2>/dev/null)
  if echo "$acl" | grep -q "AllUsers"; then
    echo "WARNING: Bucket $bucket is publicly accessible"
  fi
done

# Check for overly permissive security groups
aws ec2 describe-security-groups \
  --filters Name=ip-permission.cidr,Values=0.0.0.0/0 \
  --query 'SecurityGroups[?IpPermissions[?FromPort==`22`]].[GroupId,GroupName]' \
  --output table
```

## Troubleshooting

### IAM Permission Denied

```bash
# Decode authorization failure message
aws sts decode-authorization-message \
  --encoded-message <encoded-message> \
  --query DecodedMessage \
  --output text | jq .
```

### CloudTrail Not Logging

```bash
# Check trail status
aws cloudtrail get-trail-status --name lab-security-trail

# Verify S3 bucket policy allows CloudTrail
aws s3api get-bucket-policy --bucket lab-cloudtrail-logs-$AWS_ACCOUNT_ID

# Test permissions
aws cloudtrail start-logging --name lab-security-trail
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxxxx

# Verify security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Check NACL rules
aws ec2 describe-network-acls --network-acl-ids acl-xxxxx

# Test connectivity using VPC Reachability Analyzer
aws ec2 create-network-insights-path \
  --source <eni-id> \
  --destination <eni-id> \
  --protocol tcp \
  --destination-port 443
```

### KMS Key Access Issues

```bash
# Check key policy
aws kms get-key-policy --key-id alias/lab-key --policy-name default

# List grants on key
aws kms list-grants --key-id alias/lab-key

# Test encryption
aws kms generate-data-key --key-id alias/lab-key --key-spec AES_256
```

## Best Practices

1. **Always enable MFA** for root and privileged users
2. **Use IAM roles** instead of access keys for EC2 instances
3. **Enable CloudTrail** in all regions for audit logging
4. **Encrypt data at rest** using KMS for S3, EBS, and RDS
5. **Apply least privilege** principle to all IAM policies
6. **Use VPC endpoints** to keep traffic within AWS network
7. **Enable VPC Flow Logs** for network traffic analysis
8. **Regularly review** security groups and NACLs
9. **Set up CloudWatch alarms** for security-critical events
10. **Tag all resources** for better tracking and cost allocation

## Documentation Standards

Each lab should document:

- **Objective**: What security concept is being learned
- **Prerequisites**: Required knowledge and resources
- **Implementation Steps**: Detailed commands with explanations
- **Screenshots**: Visual proof of configuration
- **Testing**: How to verify the security control works
- **Cleanup**: Commands to remove resources (avoid charges)
- **Lessons Learned**: Key takeaways and best practices

## Environment Variables

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
export AWS_ACCOUNT_ID="123456789012"
export MY_IP="203.0.113.5"
export EMAIL_ADDRESS="alerts@example.com"
```

## Additional Resources

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [AWS IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)
- [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)
