---
name: ikb42603-cloud-security-essentials
description: Educational lab repository for AWS cloud computing security fundamentals covering IAM, encryption, VPC isolation, network security, and monitoring
triggers:
  - "help me with cloud security lab exercises"
  - "set up AWS security labs for IKB42603"
  - "configure IAM users and roles for cloud security course"
  - "implement AWS encryption and key management lab"
  - "create VPC isolation and multitenancy setup"
  - "set up CloudTrail and CloudWatch monitoring"
  - "work on cloud computing security essentials"
  - "complete AWS security lab assignments"
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS is an educational repository containing hands-on laboratory exercises for AWS cloud security fundamentals. The course covers five core security domains: account security and IAM, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

This skill helps you complete and document AWS security labs following academic best practices.

## Repository Structure

The repository follows this standard structure:

```
IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/
│
├── README.md
├── Lab0_Environment_Setup.md
├── Lab1_Account_Security_and_IAM.md
├── Lab2_Secure_Isolation_and_Multitenancy.md
├── Lab3_Encryption_and_Key_Management.md
├── Lab4_Access_Control_and_Network_Security.md
└── Lab5_Monitoring_Logging_and_Incident_Detection.md
```

## Initial Setup

### Create Repository

```bash
# Clone or create the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Initialize if creating from scratch
git init
echo "# IKB42603 - Cloud Computing Security Essentials" > README.md
git add README.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
git push -u origin main
```

### Environment Setup

Create `Lab0_Environment_Setup.md`:

```markdown
# Lab 0 - Environment Setup

## Objective
Configure AWS CLI and prepare local environment for cloud security labs.

## Prerequisites
- AWS Account (Free Tier)
- AWS CLI installed
- Git installed

## Steps

### 1. Install AWS CLI

**macOS:**
```bash
brew install awscli
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Windows:**
Download from: https://aws.amazon.com/cli/

### 2. Configure AWS Credentials

```bash
aws configure
```

Enter:
- AWS Access Key ID: `${AWS_ACCESS_KEY_ID}`
- AWS Secret Access Key: `${AWS_SECRET_ACCESS_KEY}`
- Default region: `us-east-1`
- Default output format: `json`

### 3. Verify Configuration

```bash
aws sts get-caller-identity
```

## Screenshots
![AWS CLI Configuration](./images/lab0-aws-cli-config.png)

## References
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
```

## Lab 1: Account Security and IAM

### Key Concepts
- IAM Users, Groups, and Roles
- Multi-Factor Authentication (MFA)
- Password Policies
- Access Keys Management

### Example Implementation

Create `Lab1_Account_Security_and_IAM.md`:

```markdown
# Lab 1 - Account Security and IAM

## Objective
Implement secure identity and access management practices using AWS IAM.

## Learning Outcomes
- Create IAM users and groups
- Configure password policies
- Enable MFA
- Assign permissions using policies

## Implementation

### Step 1: Create IAM User

```bash
# Create IAM user
aws iam create-user --user-name lab-admin

# Create login profile with password
aws iam create-login-profile \
  --user-name lab-admin \
  --password "${TEMP_PASSWORD}" \
  --password-reset-required
```

### Step 2: Create IAM Group

```bash
# Create administrators group
aws iam create-group --group-name Administrators

# Attach managed policy
aws iam attach-group-policy \
  --group-name Administrators \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Add user to group
aws iam add-user-to-group \
  --user-name lab-admin \
  --group-name Administrators
```

### Step 3: Configure Password Policy

```bash
# Set account password policy
aws iam update-account-password-policy \
  --minimum-password-length 12 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --max-password-age 90 \
  --password-reuse-prevention 5
```

### Step 4: Enable MFA (Console Steps)

1. Sign in to AWS Console
2. Navigate to IAM > Users > lab-admin
3. Security credentials tab
4. Click "Assign MFA device"
5. Choose Virtual MFA device
6. Scan QR code with authenticator app
7. Enter two consecutive MFA codes

### Step 5: Create Custom IAM Policy

```bash
# Create policy document
cat > s3-readonly-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:Get*",
        "s3:List*"
      ],
      "Resource": "*"
    }
  ]
}
EOF

# Create the policy
aws iam create-policy \
  --policy-name S3ReadOnlyAccess \
  --policy-document file://s3-readonly-policy.json
```

### Step 6: Create IAM Role

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

# Create role
aws iam create-role \
  --role-name EC2-S3-ReadOnly-Role \
  --assume-role-policy-document file://ec2-trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-ReadOnly-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

## Screenshots
![IAM User Created](./images/lab1-iam-user.png)
![MFA Enabled](./images/lab1-mfa-enabled.png)

## Challenges Encountered
- Initial confusion about difference between policies and roles
- MFA setup required mobile authenticator app

## Lessons Learned
- Principle of least privilege is crucial
- Always enable MFA for privileged accounts
- Use groups for permission management, not individual users

## References
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
```

## Lab 2: Secure Isolation and Multitenancy

### Key Concepts
- Virtual Private Cloud (VPC)
- Subnets (Public/Private)
- Route Tables
- Internet Gateway
- NAT Gateway

### Example Implementation

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab2-VPC}]' \
  --query 'Vpc.VpcId' \
  --output text

VPC_ID="<vpc-id-output>"

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

# Create and attach Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab2-IGW}]' \
  --query 'InternetGateway.InternetGatewayId' \
  --output text

IGW_ID="<igw-id-output>"

aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID

# Configure route table for public subnet
aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]'

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id <rtb-id> \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID
```

## Lab 3: Encryption and Key Management

### Key Concepts
- AWS Key Management Service (KMS)
- Customer Master Keys (CMK)
- S3 Encryption
- EBS Encryption

### Example Implementation

```bash
# Create KMS key
aws kms create-key \
  --description "Lab3 Customer Master Key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS \
  --query 'KeyMetadata.KeyId' \
  --output text

KEY_ID="<key-id-output>"

# Create alias for the key
aws kms create-alias \
  --alias-name alias/lab3-cmk \
  --target-key-id $KEY_ID

# Create encrypted S3 bucket
aws s3api create-bucket \
  --bucket lab3-encrypted-bucket-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket lab3-encrypted-bucket-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "'$KEY_ID'"
      }
    }]
  }'

# Upload encrypted object
echo "Sensitive data" > test-file.txt
aws s3 cp test-file.txt s3://lab3-encrypted-bucket-${AWS_ACCOUNT_ID}/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id $KEY_ID

# Verify encryption
aws s3api head-object \
  --bucket lab3-encrypted-bucket-${AWS_ACCOUNT_ID} \
  --key test-file.txt
```

## Lab 4: Access Control and Network Security

### Key Concepts
- Security Groups
- Network ACLs
- VPC Peering
- Security Group Rules

### Example Implementation

```bash
# Create security group for web server
aws ec2 create-security-group \
  --group-name WebServerSG \
  --description "Security group for web servers" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text

SG_ID="<sg-id-output>"

# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP only
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr ${MY_IP}/32

# Create security group for database
aws ec2 create-security-group \
  --group-name DatabaseSG \
  --description "Security group for database servers" \
  --vpc-id $VPC_ID

DB_SG_ID="<db-sg-id-output>"

# Allow MySQL only from web server security group
aws ec2 authorize-security-group-ingress \
  --group-id $DB_SG_ID \
  --protocol tcp \
  --port 3306 \
  --source-group $SG_ID

# Create Network ACL
aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=CustomNACL}]'

NACL_ID="<nacl-id-output>"

# Add inbound rule
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Key Concepts
- AWS CloudTrail
- Amazon CloudWatch
- CloudWatch Logs
- CloudWatch Alarms

### Example Implementation

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Create bucket policy for CloudTrail
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
      "Resource": "arn:aws:s3:::cloudtrail-logs-${AWS_ACCOUNT_ID}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::cloudtrail-logs-${AWS_ACCOUNT_ID}/*",
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
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --policy file://cloudtrail-bucket-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name Lab5-Trail \
  --s3-bucket-name cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name Lab5-Trail

# Create CloudWatch Log Group
aws logs create-log-group \
  --log-group-name /aws/security/lab5

# Create metric filter for failed login attempts
aws logs put-metric-filter \
  --log-group-name /aws/security/lab5 \
  --filter-name FailedLoginAttempts \
  --filter-pattern '[time, request_id, event_type = ConsoleLogin, event_outcome = Failure]' \
  --metric-transformations \
    metricName=FailedConsoleLogins,\
metricNamespace=SecurityMetrics,\
metricValue=1

# Create CloudWatch Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name FailedLoginAlarm \
  --alarm-description "Alert on failed login attempts" \
  --metric-name FailedConsoleLogins \
  --namespace SecurityMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1

# Query CloudTrail events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin \
  --max-results 10 \
  --query 'Events[*].[EventTime,Username,CloudTrailEvent]' \
  --output table
```

## Documentation Best Practices

### Lab Report Template

Each lab markdown file should follow this structure:

```markdown
# Lab X - [Title]

## Objective
Clear statement of what you're learning.

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Environment
- AWS Region: us-east-1
- AWS CLI Version: 2.x
- Tools: AWS Console, AWS CLI

## Implementation

### Step 1: [Task Name]

Description of what you're doing.

```bash
# Commands with comments
aws service action --parameter value
```

**Expected Output:**
```
Output example
```

### Step 2: [Next Task]

Continue with clear steps...

## Commands Reference

| Command | Purpose |
|---------|---------|
| `aws iam create-user` | Create IAM user |
| `aws ec2 describe-vpcs` | List VPCs |

## Screenshots

![Description](./images/labX-screenshot1.png)
![Another view](./images/labX-screenshot2.png)

## Challenges Encountered

1. **Challenge:** Brief description
   **Solution:** How you resolved it

## Lessons Learned

- Key takeaway 1
- Key takeaway 2

## Security Best Practices Applied

- Practice 1
- Practice 2

## References

- [AWS Documentation](https://docs.aws.amazon.com/)
- Course materials
```

## Git Workflow

### Regular Commits

```bash
# Check status
git status

# Add files
git add Lab1_Account_Security_and_IAM.md
git add images/

# Commit with meaningful message
git commit -m "Complete Lab 1: IAM users, groups, and MFA setup"

# Push to GitHub
git push origin main
```

### Commit Message Best Practices

```bash
# Good commit messages
git commit -m "Add Lab 1 IAM configuration screenshots"
git commit -m "Complete Step 3: Configure password policy"
git commit -m "Fix typo in Lab 2 VPC documentation"
git commit -m "Add troubleshooting section to Lab 3"

# Bad commit messages (avoid)
git commit -m "update"
git commit -m "lab done"
git commit -m "changes"
```

## Common Patterns

### Creating Lab Documentation

```bash
#!/bin/bash
# Script to create new lab file

LAB_NUMBER=$1
LAB_TITLE=$2

cat > Lab${LAB_NUMBER}_${LAB_TITLE}.md <<EOF
# Lab ${LAB_NUMBER} - ${LAB_TITLE}

## Objective


## Learning Outcomes


## Implementation

### Step 1:


## Screenshots


## Challenges Encountered


## Lessons Learned


## References

EOF

echo "Created Lab${LAB_NUMBER}_${LAB_TITLE}.md"
```

### Screenshot Organization

```bash
# Create images directory structure
mkdir -p images/lab{1..5}

# Naming convention
# images/lab1/step1-iam-user-creation.png
# images/lab1/step2-mfa-enabled.png
# images/lab2/vpc-diagram.png
```

## Troubleshooting

### AWS CLI Not Configured

**Problem:** `Unable to locate credentials`

**Solution:**
```bash
# Configure credentials
aws configure

# Or use environment variables
export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
export AWS_DEFAULT_REGION=us-east-1
```

### Permission Denied Errors

**Problem:** `AccessDenied` when running AWS commands

**Solution:**
```bash
# Check current identity
aws sts get-caller-identity

# Verify IAM permissions
aws iam get-user

# Ensure your user has necessary policies attached
```

### Git Push Rejected

**Problem:** `Updates were rejected because the remote contains work that you do not have locally`

**Solution:**
```bash
# Pull remote changes first
git pull origin main

# If conflicts occur, resolve them
git status
# Edit conflicting files
git add .
git commit -m "Resolve merge conflicts"
git push origin main
```

### Region-Specific Resources

**Problem:** Resources not found in different regions

**Solution:**
```bash
# Always specify region
aws ec2 describe-instances --region us-east-1

# Or set default region
aws configure set region us-east-1
```

## AWS CLI Cheat Sheet

### IAM Commands

```bash
# List users
aws iam list-users

# List groups
aws iam list-groups

# List policies
aws iam list-policies --scope Local

# Get user details
aws iam get-user --user-name username
```

### EC2/VPC Commands

```bash
# List VPCs
aws ec2 describe-vpcs

# List subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-xxx"

# List security groups
aws ec2 describe-security-groups

# List instances
aws ec2 describe-instances
```

### S3 Commands

```bash
# List buckets
aws s3 ls

# List bucket contents
aws s3 ls s3://bucket-name/

# Copy file to S3
aws s3 cp file.txt s3://bucket-name/

# Get bucket encryption
aws s3api get-bucket-encryption --bucket bucket-name
```

### KMS Commands

```bash
# List keys
aws kms list-keys

# Describe key
aws kms describe-key --key-id key-id

# List aliases
aws kms list-aliases
```

### CloudTrail Commands

```bash
# List trails
aws cloudtrail describe-trails

# Get trail status
aws cloudtrail get-trail-status --name trail-name

# Lookup events
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
```

## Academic Integrity Guidelines

- **DO:** Learn and understand each command
- **DO:** Customize configurations for your environment
- **DO:** Document your own challenges and solutions
- **DO:** Take your own screenshots
- **DON'T:** Copy lab reports verbatim
- **DON'T:** Share AWS credentials
- **DON'T:** Submit others' work as your own

## Additional Resources

- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)
- [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/)
