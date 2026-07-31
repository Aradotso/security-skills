---
name: ikb42603-cloud-security-essentials
description: Educational lab repository for AWS cloud computing security fundamentals including IAM, encryption, network security, and monitoring
triggers:
  - how do I set up AWS cloud security labs
  - show me IAM security best practices
  - help with AWS security group configuration
  - configure AWS KMS encryption
  - set up CloudTrail and CloudWatch monitoring
  - implement AWS VPC network security
  - create secure multi-tenant AWS architecture
  - AWS security lab exercises
---

# IKB42603 Cloud Computing Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS is an educational repository containing hands-on AWS security laboratory exercises. It covers five core areas of cloud security: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

This skill helps you navigate and implement AWS security best practices across identity management, network isolation, encryption, access control, and security monitoring.

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

## Setup

### Prerequisites

- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Git installed
- Basic understanding of cloud computing concepts

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git

# Navigate to project directory
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Configure AWS CLI (if not already done)
aws configure
# Enter AWS_ACCESS_KEY_ID when prompted
# Enter AWS_SECRET_ACCESS_KEY when prompted
# Enter default region (e.g., us-east-1)
# Enter output format (json recommended)
```

### Verify AWS Configuration

```bash
# Check AWS identity
aws sts get-caller-identity

# List available regions
aws ec2 describe-regions --output table
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-01

# Create access keys for the user
aws iam create-access-key --user-name lab-user-01

# Attach a policy to the user
aws iam attach-user-policy \
  --user-name lab-user-01 \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Creating IAM Groups

```bash
# Create an IAM group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-01 \
  --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

### Creating Custom IAM Policies

```bash
# Create policy document (save as policy.json)
cat > policy.json << 'EOF'
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
        "arn:aws:s3:::my-secure-bucket",
        "arn:aws:s3:::my-secure-bucket/*"
      ]
    }
  ]
}
EOF

# Create the policy
aws iam create-policy \
  --policy-name S3ReadOnlyCustom \
  --policy-document file://policy.json
```

### Enabling MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-01-mfa \
  --outfile /tmp/QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA device (requires two consecutive authentication codes)
aws iam enable-mfa-device \
  --user-name lab-user-01 \
  --serial-number arn:aws:iam::ACCOUNT_ID:mfa/lab-user-01-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### Password Policy

```bash
# Set account password policy
aws iam update-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --max-password-age 90 \
  --password-reuse-prevention 5
```

## Lab 2: Secure Isolation and Multitenancy

### Creating VPC for Isolation

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=LabVPC}]'

# Store VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=LabVPC" \
  --query "Vpcs[0].VpcId" \
  --output text)

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
```

### Creating Internet Gateway

```bash
# Create internet gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=LabIGW}]'

IGW_ID=$(aws ec2 describe-internet-gateways \
  --filters "Name=tag:Name,Values=LabIGW" \
  --query "InternetGateways[0].InternetGatewayId" \
  --output text)

# Attach to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
```

### Configuring Route Tables

```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]'

RTB_ID=$(aws ec2 describe-route-tables \
  --filters "Name=tag:Name,Values=PublicRouteTable" \
  --query "RouteTables[0].RouteTableId" \
  --output text)

# Add route to internet gateway
aws ec2 create-route \
  --route-table-id $RTB_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associate with public subnet
SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=PublicSubnet" \
  --query "Subnets[0].SubnetId" \
  --output text)

aws ec2 associate-route-table \
  --route-table-id $RTB_ID \
  --subnet-id $SUBNET_ID
```

## Lab 3: Encryption and Key Management

### Creating KMS Keys

```bash
# Create customer managed key
aws kms create-key \
  --description "Lab encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Store key ID
KEY_ID=$(aws kms list-keys \
  --query "Keys[0].KeyId" \
  --output text)

# Create alias
aws kms create-alias \
  --alias-name alias/lab-key \
  --target-key-id $KEY_ID
```

### Encrypting Data with KMS

```bash
# Encrypt plaintext
echo "Sensitive data" > plaintext.txt

aws kms encrypt \
  --key-id alias/lab-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob > encrypted.txt

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.txt \
  --output text \
  --query Plaintext | base64 --decode
```

### S3 Bucket Encryption

```bash
# Create S3 bucket with encryption
aws s3api create-bucket \
  --bucket my-secure-lab-bucket-$(date +%s) \
  --region us-east-1

BUCKET_NAME="my-secure-lab-bucket-<timestamp>"

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab-key"
      }
    }]
  }'

# Upload encrypted object
aws s3 cp plaintext.txt s3://$BUCKET_NAME/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id alias/lab-key
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
```

## Lab 4: Access Control and Network Security

### Security Group Configuration

```bash
# Create security group for web server
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Security group for web servers" \
  --vpc-id $VPC_ID

SG_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=web-server-sg" \
  --query "SecurityGroups[0].GroupId" \
  --output text)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24
```

### Network ACL Configuration

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=LabNACL}]'

NACL_ID=$(aws ec2 describe-network-acls \
  --filters "Name=tag:Name,Values=LabNACL" \
  --query "NetworkAcls[0].NetworkAclId" \
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

# Allow outbound traffic
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --egress true \
  --rule-action allow
```

### S3 Bucket Policy

```bash
# Create bucket policy for access control
cat > bucket-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::BUCKET_NAME/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    },
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME",
        "arn:aws:s3:::BUCKET_NAME/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
EOF

# Replace BUCKET_NAME and apply policy
sed -i "s/BUCKET_NAME/$BUCKET_NAME/g" bucket-policy.json
aws s3api put-bucket-policy \
  --bucket $BUCKET_NAME \
  --policy file://bucket-policy.json
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Enabling CloudTrail

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-$(aws sts get-caller-identity --query Account --output text) \
  --region us-east-1

TRAIL_BUCKET="cloudtrail-logs-$(aws sts get-caller-identity --query Account --output text)"

# Create CloudTrail
aws cloudtrail create-trail \
  --name lab-trail \
  --s3-bucket-name $TRAIL_BUCKET \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name lab-trail

# Verify trail status
aws cloudtrail get-trail-status --name lab-trail
```

### CloudWatch Logs Configuration

```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lab/security

# Create log stream
aws logs create-log-stream \
  --log-group-name /aws/lab/security \
  --log-stream-name security-events

# Put log events
aws logs put-log-events \
  --log-group-name /aws/lab/security \
  --log-stream-name security-events \
  --log-events timestamp=$(date +%s000),message="Security event logged"
```

### CloudWatch Alarms

```bash
# Create alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICalls \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold
```

### CloudWatch Metric Filters

```bash
# Create metric filter for failed login attempts
aws logs put-metric-filter \
  --log-group-name /aws/lab/security \
  --filter-name FailedLoginAttempts \
  --filter-pattern '[time, request_id, event_type = ConsoleLogin, result = Failed, ...]' \
  --metric-transformations \
    metricName=FailedLoginAttempts,metricNamespace=SecurityMetrics,metricValue=1
```

### GuardDuty Configuration

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

# Get detector ID
DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)

# List findings
aws guardduty list-findings --detector-id $DETECTOR_ID

# Get finding details
aws guardduty get-findings \
  --detector-id $DETECTOR_ID \
  --finding-ids <finding-id>
```

## Common Patterns

### Secure EC2 Instance Launch

```bash
#!/bin/bash
# Launch EC2 instance with security best practices

# Variables
VPC_ID="vpc-xxxxx"
SUBNET_ID="subnet-xxxxx"
SG_ID="sg-xxxxx"
KEY_NAME="lab-key"
AMI_ID="ami-xxxxx"  # Amazon Linux 2 AMI

# Create encrypted key pair
aws ec2 create-key-pair \
  --key-name $KEY_NAME \
  --query 'KeyMaterial' \
  --output text > ~/.ssh/$KEY_NAME.pem
chmod 400 ~/.ssh/$KEY_NAME.pem

# Launch instance with encrypted EBS
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.micro \
  --key-name $KEY_NAME \
  --security-group-ids $SG_ID \
  --subnet-id $SUBNET_ID \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/xvda",
      "Ebs": {
        "VolumeSize": 8,
        "VolumeType": "gp3",
        "Encrypted": true,
        "KmsKeyId": "alias/lab-key"
      }
    }
  ]' \
  --metadata-options 'HttpTokens=required,HttpPutResponseHopLimit=1' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=SecureInstance}]'
```

### S3 Bucket Security Hardening

```bash
#!/bin/bash
# Harden S3 bucket security

BUCKET_NAME="secure-bucket-$(date +%s)"

# Create bucket
aws s3api create-bucket --bucket $BUCKET_NAME --region us-east-1

# Block public access
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled

# Enable logging
aws s3api put-bucket-logging \
  --bucket $BUCKET_NAME \
  --bucket-logging-status '{
    "LoggingEnabled": {
      "TargetBucket": "'"$TRAIL_BUCKET"'",
      "TargetPrefix": "s3-logs/"
    }
  }'

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab-key"
      },
      "BucketKeyEnabled": true
    }]
  }'
```

### IAM Role for Lambda with Least Privilege

```bash
# Create trust policy
cat > lambda-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "lambda.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Create role
aws iam create-role \
  --role-name LambdaSecureRole \
  --assume-role-policy-document file://lambda-trust-policy.json

# Create custom policy
cat > lambda-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::specific-bucket/*"
    }
  ]
}
EOF

# Attach policy
aws iam put-role-policy \
  --role-name LambdaSecureRole \
  --policy-name LambdaSecurePolicy \
  --policy-document file://lambda-policy.json
```

## Troubleshooting

### AWS CLI Authentication Issues

```bash
# Check current credentials
aws sts get-caller-identity

# List configured profiles
aws configure list-profiles

# Use specific profile
aws s3 ls --profile lab-profile

# Check credential file
cat ~/.aws/credentials

# Check config file
cat ~/.aws/config
```

### IAM Permission Denied

```bash
# Simulate policy to check permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT_ID:user/username \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::bucket-name/object-key

# Get attached policies for user
aws iam list-attached-user-policies --user-name username

# Get inline policies
aws iam list-user-policies --user-name username

# Get policy version
aws iam get-policy-version \
  --policy-arn arn:aws:iam::aws:policy/PolicyName \
  --version-id v1
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID"

# Check security group rules
aws ec2 describe-security-groups --group-ids $SG_ID

# Check NACL rules
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=$VPC_ID"

# Test connectivity from EC2 instance (via Systems Manager)
aws ssm send-command \
  --instance-ids i-xxxxx \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["ping -c 4 8.8.8.8"]'
```

### KMS Key Access Denied

```bash
# Check key policy
aws kms get-key-policy \
  --key-id alias/lab-key \
  --policy-name default

# List grants on key
aws kms list-grants --key-id alias/lab-key

# Check if key is enabled
aws kms describe-key --key-id alias/lab-key
```

### CloudTrail Not Logging

```bash
# Verify trail status
aws cloudtrail get-trail-status --name lab-trail

# Check S3 bucket policy
aws s3api get-bucket-policy --bucket $TRAIL_BUCKET

# Validate trail configuration
aws cloudtrail describe-trails --trail-name-list lab-trail

# Check event selectors
aws cloudtrail get-event-selectors --trail-name lab-trail
```

## Environment Variables

Always use environment variables for sensitive data:

```bash
# Set AWS credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-east-1

# Set in profile
aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID --profile lab
aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY --profile lab
aws configure set region us-east-1 --profile lab
```

## Documentation Standards

When documenting lab work, include:

```markdown
## Lab X: [Lab Title]

### Objective
Brief description of what this lab accomplishes

### Prerequisites
- List required resources
- Required permissions
- Prerequisite labs

### Implementation Steps

#### Step 1: [Action]
```bash
# Command with explanation
aws service action --parameters
```

**Expected Output:**
```
Sample output
```

**Screenshot:**
![Description](path/to/screenshot.png)

#### Step 2: [Next Action]
Continue pattern...

### Verification
```bash
# Commands to verify implementation
```

### Cleanup
```bash
# Commands to remove resources
```

### Lessons Learned
- Key takeaway 1
- Key takeaway 2

### References
- Documentation links
- Relevant AWS documentation
```

## Git Workflow

```bash
# Check status
git status

# Add specific lab files
git add Lab1_Account_Security_and_IAM.md
git add screenshots/

# Commit with descriptive message
git commit -m "Complete Lab 1: IAM users, groups, and MFA configuration"

# Push to GitHub
git push origin main

# Create feature branch for experimentation
git checkout -b lab2-vpc-experiments
git push origin lab2-vpc-experiments
```

## Best Practices

1. **Never commit credentials**: Use AWS CLI profiles and environment variables
2. **Document everything**: Include command explanations and screenshots
3. **Test in isolated environment**: Use separate AWS accounts or VPCs for labs
4. **Clean up resources**: Always delete resources after completing labs to avoid charges
5. **Version control**: Commit regularly with meaningful messages
6. **Use tags**: Tag all AWS resources for easy identification and cleanup
7. **Enable billing alerts**: Set up CloudWatch billing alarms to avoid unexpected charges
8. **Follow least privilege**: Grant minimum necessary permissions
9. **Enable MFA**: Always enable MFA for root and IAM users
10. **Encrypt everything**: Use KMS for encryption at rest, TLS for encryption in transit
