---
name: ikb42603-cloud-security-essentials
description: Educational AWS cloud security lab environment covering IAM, encryption, network security, and monitoring with hands-on exercises
triggers:
  - how do I set up AWS security labs
  - show me cloud security best practices
  - help me configure IAM policies and roles
  - how to implement AWS encryption and KMS
  - guide me through AWS security monitoring
  - what are AWS VPC security best practices
  - help with CloudTrail and CloudWatch setup
  - show me AWS security group configuration
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603 Cloud Computing Security Essentials is an educational repository providing hands-on AWS security labs covering five core areas: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection. This skill helps you implement AWS security best practices through structured laboratory exercises.

## Repository Structure

The course is organized into 5 weekly labs:

- **Lab 1**: Account Security and IAM
- **Lab 2**: Secure Isolation and Multitenancy
- **Lab 3**: Encryption and Key Management
- **Lab 4**: Access Control and Network Security
- **Lab 5**: Monitoring, Logging, and Incident Detection

## Installation & Setup

### Initial Repository Setup

```bash
# Clone the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create lab structure
mkdir -p Lab{1..5}
touch Lab0_Environment_Setup.md
touch Lab{1..5}_*.md
touch README.md
```

### AWS CLI Setup

```bash
# Install AWS CLI (macOS)
brew install awscli

# Install AWS CLI (Linux)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS credentials
aws configure
# Enter: AWS Access Key ID, Secret Access Key, Default region, Output format
```

### Environment Variables

```bash
# Set AWS credentials via environment variables
export AWS_ACCESS_KEY_ID=<your-access-key>
export AWS_SECRET_ACCESS_KEY=<your-secret-key>
export AWS_DEFAULT_REGION=us-east-1
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-01

# Create access key for programmatic access
aws iam create-access-key --user-name lab-user-01

# Attach managed policy to user
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Custom IAM Policy

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
# Create custom policy from JSON file
aws iam create-policy \
  --policy-name Lab-S3-ReadOnly \
  --policy-document file://s3-readonly-policy.json

# Attach custom policy to user
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::123456789012:policy/Lab-S3-ReadOnly
```

### Enable MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-mfa-device \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA device for user (requires two consecutive codes)
aws iam enable-mfa-device \
  --user-name lab-user-01 \
  --serial-number arn:aws:iam::123456789012:mfa/lab-mfa-device \
  --authentication-code1 123456 \
  --authentication-code2 789012
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
  --role-name Lab-EC2-S3-Access \
  --assume-role-policy-document file://ec2-trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name Lab-EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name Lab-EC2-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name Lab-EC2-Profile \
  --role-name Lab-EC2-S3-Access
```

## Lab 2: Secure Isolation and Multitenancy

### VPC Creation

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab-VPC}]'

# Create public subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxxxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Lab-Public-Subnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxxxxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Lab-Private-Subnet}]'

# Create internet gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab-IGW}]'

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxxxxxxxx \
  --vpc-id vpc-xxxxxxxxx
```

### Security Groups

```bash
# Create security group
aws ec2 create-security-group \
  --group-name Lab-Web-SG \
  --description "Security group for web servers" \
  --vpc-id vpc-xxxxxxxxx

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24
```

### Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-xxxxxxxxx \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Lab-NACL}]'

# Allow inbound HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxxxxxx \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Allow outbound traffic
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxxxxxx \
  --egress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow
```

## Lab 3: Encryption and Key Management

### AWS KMS Key Creation

```bash
# Create KMS key
aws kms create-key \
  --description "Lab encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create alias for key
aws kms create-alias \
  --alias-name alias/lab-key \
  --target-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012

# List KMS keys
aws kms list-keys

# Describe key
aws kms describe-key --key-id alias/lab-key
```

### S3 Bucket Encryption

```bash
# Create S3 bucket
aws s3api create-bucket \
  --bucket lab-encrypted-bucket-$(date +%s) \
  --region us-east-1

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket lab-encrypted-bucket-123456789 \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab-key"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Verify encryption configuration
aws s3api get-bucket-encryption \
  --bucket lab-encrypted-bucket-123456789
```

### Encrypt/Decrypt Data with KMS

```bash
# Encrypt plaintext
echo "Sensitive data" > plaintext.txt
aws kms encrypt \
  --key-id alias/lab-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.dat

# Decrypt ciphertext
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.dat \
  --output text \
  --query Plaintext | base64 --decode
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
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=Lab-Encrypted-Volume}]'

# Launch EC2 instance with encrypted root volume
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --block-device-mappings '[{
    "DeviceName": "/dev/xvda",
    "Ebs": {
      "VolumeSize": 8,
      "VolumeType": "gp3",
      "Encrypted": true,
      "KmsKeyId": "alias/lab-key"
    }
  }]'
```

## Lab 4: Access Control and Network Security

### VPC Flow Logs

```bash
# Create CloudWatch log group
aws logs create-log-group \
  --log-group-name /aws/vpc/flowlogs

# Create IAM role for Flow Logs
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

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxxxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/VPCFlowLogsRole
```

### AWS WAF Configuration

```bash
# Create IP set
aws wafv2 create-ip-set \
  --name Lab-Blocked-IPs \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses 192.0.2.0/24 203.0.113.0/24

# Create web ACL
aws wafv2 create-web-acl \
  --name Lab-WebACL \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://waf-rules.json \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=LabWebACL
```

### S3 Bucket Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::lab-bucket-name",
        "arn:aws:s3:::lab-bucket-name/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    },
    {
      "Sid": "AllowSpecificVPCOnly",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::lab-bucket-name/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpc": "vpc-xxxxxxxxx"
        }
      }
    }
  ]
}
```

```bash
# Apply bucket policy
aws s3api put-bucket-policy \
  --bucket lab-bucket-name \
  --policy file://bucket-policy.json

# Block public access
aws s3api put-public-access-block \
  --bucket lab-bucket-name \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket lab-cloudtrail-logs-$(date +%s) \
  --region us-east-1

# Create trail
aws cloudtrail create-trail \
  --name Lab-Trail \
  --s3-bucket-name lab-cloudtrail-logs-123456789 \
  --is-multi-region-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging --name Lab-Trail

# Verify trail status
aws cloudtrail get-trail-status --name Lab-Trail

# Lookup recent events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser \
  --max-results 10
```

### CloudWatch Alarms

```bash
# Create SNS topic for alerts
aws sns create-topic --name Lab-Security-Alerts

# Subscribe email to topic
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:Lab-Security-Alerts \
  --protocol email \
  --notification-endpoint ${ALERT_EMAIL}

# Create CloudWatch alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICalls \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:Lab-Security-Alerts
```

### CloudWatch Log Insights Queries

```bash
# Query CloudTrail logs for failed console logins
aws logs start-query \
  --log-group-name /aws/cloudtrail/logs \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string '
    fields @timestamp, userIdentity.userName, errorCode, errorMessage
    | filter eventName = "ConsoleLogin" and errorCode exists
    | sort @timestamp desc
    | limit 20
  '

# Query VPC Flow Logs for rejected connections
aws logs start-query \
  --log-group-name /aws/vpc/flowlogs \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string '
    fields @timestamp, srcAddr, dstAddr, dstPort, action
    | filter action = "REJECT"
    | stats count(*) by srcAddr
    | sort count desc
    | limit 20
  '
```

### AWS Config Rules

```bash
# Enable AWS Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig \
  --recording-group allSupported=true,includeGlobalResourceTypes=true

# Create Config rule for S3 bucket encryption
aws configservice put-config-rule \
  --config-rule file://s3-encryption-rule.json

# Example rule JSON
cat > s3-encryption-rule.json <<EOF
{
  "ConfigRuleName": "s3-bucket-server-side-encryption-enabled",
  "Description": "Checks that S3 buckets have encryption enabled",
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
  }
}
EOF
```

### GuardDuty Setup

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

# Get detector ID
DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)

# List findings
aws guardduty list-findings \
  --detector-id $DETECTOR_ID \
  --finding-criteria '{"Criterion":{"severity":{"Gte":7}}}'

# Get finding details
aws guardduty get-findings \
  --detector-id $DETECTOR_ID \
  --finding-ids <finding-id>
```

## Common Patterns

### Lab Documentation Template

```markdown
# Lab X: [Lab Title]

## Objective
[Clear statement of what this lab aims to achieve]

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Environment
- AWS Region: us-east-1
- Services: IAM, EC2, S3, etc.

## Step-by-Step Implementation

### Step 1: [Action]
```bash
# Command with explanation
aws service command --parameter value
```

**Screenshot**: `lab-x-step-1.png`

**Explanation**: [Why this step is important]

### Step 2: [Next Action]
[Continue pattern]

## Commands Summary
```bash
# All commands used in order
```

## Challenges Encountered
- Challenge 1: [Description and solution]
- Challenge 2: [Description and solution]

## Lessons Learned
- Key takeaway 1
- Key takeaway 2

## References
- [AWS Documentation Link]
- [Security Best Practices Link]
```

### Git Workflow

```bash
# Start new lab
git checkout -b lab-3-encryption
mkdir Lab3_Encryption_and_Key_Management
cd Lab3_Encryption_and_Key_Management

# Add lab files
touch README.md
mkdir screenshots

# Regular commits
git add .
git commit -m "Lab 3: Create KMS key and enable encryption"

# Add more work
git add .
git commit -m "Lab 3: Configure S3 bucket encryption"

# Complete lab
git add .
git commit -m "Lab 3: Complete encryption lab with documentation"

# Merge to main
git checkout main
git merge lab-3-encryption
git push origin main
```

### AWS CLI Configuration Profiles

```bash
# Configure multiple profiles
aws configure --profile lab-admin
aws configure --profile lab-readonly

# Use specific profile
aws s3 ls --profile lab-admin
aws ec2 describe-instances --profile lab-readonly

# Set default profile
export AWS_PROFILE=lab-admin
```

## Troubleshooting

### IAM Permission Issues

```bash
# Check current identity
aws sts get-caller-identity

# Simulate IAM policy
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/lab-user \
  --action-names s3:GetObject s3:PutObject \
  --resource-arns arn:aws:s3:::lab-bucket/*

# Test AssumeRole
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/Lab-Role \
  --role-session-name test-session
```

### Network Connectivity Issues

```bash
# Test VPC connectivity
aws ec2 describe-vpc-peering-connections
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-xxxxxxxxx"

# Check security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx

# Test instance connectivity
aws ec2-instance-connect send-ssh-public-key \
  --instance-id i-xxxxxxxxx \
  --instance-os-user ec2-user \
  --ssh-public-key file://~/.ssh/id_rsa.pub
```

### CloudTrail Event History

```bash
# Find who made a change
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceName,AttributeValue=lab-bucket-name \
  --max-results 50

# Find events by user
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=lab-user-01

# Export events to file
aws cloudtrail lookup-events \
  --start-time $(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%S) \
  --output json > cloudtrail-events.json
```

### KMS Key Issues

```bash
# Check key state
aws kms describe-key --key-id alias/lab-key

# List grants on key
aws kms list-grants --key-id alias/lab-key

# Enable disabled key
aws kms enable-key --key-id alias/lab-key

# Rotate key
aws kms enable-key-rotation --key-id alias/lab-key
```

### S3 Access Denied

```bash
# Check bucket policy
aws s3api get-bucket-policy --bucket lab-bucket-name

# Check bucket ACL
aws s3api get-bucket-acl --bucket lab-bucket-name

# Check public access block
aws s3api get-public-access-block --bucket lab-bucket-name

# Verify object ownership
aws s3api get-object-attributes \
  --bucket lab-bucket-name \
  --key object-key \
  --object-attributes "ObjectParts"
```

## Best Practices

### Security Checklist

- [ ] Enable MFA for all IAM users
- [ ] Use IAM roles instead of access keys when possible
- [ ] Apply principle of least privilege to all policies
- [ ] Enable CloudTrail in all regions
- [ ] Enable VPC Flow Logs
- [ ] Encrypt all data at rest using KMS
- [ ] Use VPC endpoints for AWS services
- [ ] Enable GuardDuty for threat detection
- [ ] Configure AWS Config for compliance monitoring
- [ ] Set up CloudWatch alarms for security events
- [ ] Regularly rotate credentials and keys
- [ ] Use security groups as firewalls
- [ ] Enable S3 bucket versioning and logging
- [ ] Implement backup and disaster recovery

### Cost Management

```bash
# Tag resources for cost tracking
aws ec2 create-tags \
  --resources i-xxxxxxxxx \
  --tags Key=Project,Value=IKB42603 Key=Lab,Value=Lab1

# Set up billing alerts
aws cloudwatch put-metric-alarm \
  --alarm-name Lab-Budget-Alert \
  --alarm-description "Alert when estimated charges exceed $10" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --evaluation-periods 1 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold

# Clean up resources after lab
aws ec2 terminate-instances --instance-ids i-xxxxxxxxx
aws ec2 delete-security-group --group-id sg-xxxxxxxxx
aws s3 rb s3://lab-bucket-name --force
```
