---
name: ikb42603-cloud-computing-security-essentials
description: Educational AWS cloud security labs covering IAM, encryption, network security, monitoring, and incident detection
triggers:
  - how do I set up AWS IAM security labs
  - configure cloud security exercises with AWS
  - implement AWS encryption and key management practices
  - set up AWS CloudTrail and CloudWatch monitoring
  - create secure AWS VPC and network isolation
  - practice AWS security best practices in labs
  - configure AWS multi-tenancy security
  - implement AWS access control and security groups
---

# IKB42603 Cloud Computing Security Essentials

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This is an educational repository containing hands-on AWS cloud security laboratory exercises. The labs cover five core areas of cloud security: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

Each lab provides practical exercises to implement AWS security controls, understand security configurations, and develop secure cloud architectures.

## Repository Structure

The repository follows a weekly lab structure:

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

## Initial Setup

### Clone the Repository

```bash
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS
```

### Prerequisites

- AWS Account (Free Tier eligible)
- AWS CLI installed and configured
- Git for version control
- Basic understanding of cloud computing concepts

### Configure AWS CLI

```bash
# Configure AWS credentials
aws configure
# Enter your AWS Access Key ID, Secret Access Key, region, and output format
```

## Lab 1: Account Security and IAM

### Key Concepts

- AWS Identity and Access Management (IAM)
- Multi-Factor Authentication (MFA)
- Least Privilege Principle
- IAM Users, Groups, and Roles
- IAM Policies

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-1

# Create access key for the user
aws iam create-access-key --user-name lab-user-1

# Attach a policy to the user
aws iam attach-user-policy \
  --user-name lab-user-1 \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Creating IAM Groups

```bash
# Create an IAM group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-1 \
  --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

### Creating Custom IAM Policy

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
        "arn:aws:s3:::lab-bucket-name",
        "arn:aws:s3:::lab-bucket-name/*"
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
```

### Enabling MFA (Console-based)

MFA must be configured through the AWS Console for the root account and IAM users. Document the steps with screenshots.

## Lab 2: Secure Isolation and Multitenancy

### Creating a VPC

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=LabVPC}]'

# Create public subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxxxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxxxxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'
```

### Creating Internet Gateway

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=LabIGW}]'

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxxxxxxxx \
  --vpc-id vpc-xxxxxxxxx
```

### Configuring Route Tables

```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-xxxxxxxxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]'

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id rtb-xxxxxxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxxxxxxxx

# Associate route table with subnet
aws ec2 associate-route-table \
  --route-table-id rtb-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx
```

## Lab 3: Encryption and Key Management

### Creating KMS Keys

```bash
# Create a Customer Master Key (CMK)
aws kms create-key \
  --description "Lab encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create an alias for the key
aws kms create-alias \
  --alias-name alias/lab-key \
  --target-key-id <key-id>

# List KMS keys
aws kms list-keys
```

### Encrypting Data with KMS

```bash
# Encrypt data
echo "Sensitive lab data" > plaintext.txt
aws kms encrypt \
  --key-id alias/lab-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode > decrypted.txt
```

### S3 Bucket Encryption

```bash
# Create S3 bucket with encryption
aws s3api create-bucket \
  --bucket lab-encrypted-bucket-$(date +%s) \
  --region us-east-1

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket lab-encrypted-bucket-$(date +%s) \
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
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/lab-key \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EncryptedVolume}]'
```

## Lab 4: Access Control and Network Security

### Creating Security Groups

```bash
# Create security group for web servers
aws ec2 create-security-group \
  --group-name WebServerSG \
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
  --cidr ${YOUR_IP}/32
```

### Network ACLs

```bash
# Create Network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-xxxxxxxxx \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=LabNACL}]'

# Add inbound rule (allow HTTP)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxxxxxx \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Add outbound rule (allow all)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxxxxxx \
  --egress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow
```

### Launching EC2 Instance with Security

```bash
# Launch instance with security group
aws ec2 run-instances \
  --image-id ami-xxxxxxxxx \
  --instance-type t2.micro \
  --key-name lab-key-pair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=LabWebServer}]'
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Enabling CloudTrail

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-$(date +%s) \
  --region us-east-1

# Create CloudTrail
aws cloudtrail create-trail \
  --name lab-trail \
  --s3-bucket-name cloudtrail-logs-$(date +%s)

# Start logging
aws cloudtrail start-logging --name lab-trail

# Get trail status
aws cloudtrail get-trail-status --name lab-trail
```

### CloudWatch Alarms

```bash
# Create CloudWatch alarm for EC2 CPU
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu-usage \
  --alarm-description "Alarm when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-xxxxxxxxx
```

### CloudWatch Logs

```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lab/application

# Create log stream
aws logs create-log-stream \
  --log-group-name /aws/lab/application \
  --log-stream-name instance-logs

# Query logs
aws logs filter-log-events \
  --log-group-name /aws/lab/application \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s)000
```

### AWS Config for Compliance

```bash
# Create configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=lab-config,roleARN=arn:aws:iam::ACCOUNT-ID:role/config-role

# Start configuration recorder
aws configservice start-configuration-recorder \
  --configuration-recorder-name lab-config

# Create delivery channel
aws configservice put-delivery-channel \
  --delivery-channel name=lab-delivery,s3BucketName=config-bucket-name
```

## Common Patterns

### Lab Documentation Template

Each lab should include:

```markdown
## Lab X: [Title]

### Objective
[What you will learn]

### Prerequisites
- AWS Account configured
- AWS CLI installed
- [Other requirements]

### Step-by-Step Implementation

#### Step 1: [Task Name]
[Explanation]

```bash
# Commands here
```

#### Step 2: [Task Name]
[Explanation]

### Screenshots
![Description](path/to/screenshot.png)

### Challenges Encountered
- [Issue 1 and resolution]
- [Issue 2 and resolution]

### Lessons Learned
- [Key takeaway 1]
- [Key takeaway 2]

### References
- [AWS Documentation links]
```

### Git Workflow

```bash
# Check status
git status

# Add changes
git add .

# Commit with meaningful message
git commit -m "Complete Lab X: [Description]"

# Push to repository
git push origin main
```

## Troubleshooting

### AWS CLI Access Denied

```bash
# Check current identity
aws sts get-caller-identity

# Verify credentials are configured
cat ~/.aws/credentials

# Reconfigure if needed
aws configure
```

### Security Group Not Working

```bash
# Describe security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx

# Check if rules are properly configured
# Ensure VPC and subnet associations are correct
```

### CloudTrail Not Logging

```bash
# Verify trail status
aws cloudtrail get-trail-status --name lab-trail

# Check S3 bucket policy allows CloudTrail
aws s3api get-bucket-policy --bucket cloudtrail-logs-bucket-name

# Ensure trail is started
aws cloudtrail start-logging --name lab-trail
```

### KMS Encryption Failures

```bash
# List key policies
aws kms get-key-policy \
  --key-id alias/lab-key \
  --policy-name default

# Verify you have permission to use the key
aws kms describe-key --key-id alias/lab-key
```

## Best Practices

1. **Always use IAM roles** instead of embedding credentials
2. **Enable MFA** on root and privileged accounts
3. **Apply least privilege principle** to all IAM policies
4. **Enable encryption at rest** for all data stores
5. **Use Security Groups as firewalls** with default deny
6. **Enable CloudTrail logging** in all regions
7. **Set up CloudWatch alarms** for security events
8. **Regular security audits** using AWS Config
9. **Tag all resources** for better organization and cost tracking
10. **Document everything** with screenshots and explanations

## Environment Variables

Reference AWS credentials through environment variables:

```bash
export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
```

Never hardcode credentials in scripts or code.

## Additional Resources

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [AWS KMS Documentation](https://docs.aws.amazon.com/kms/)
- [AWS CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)
