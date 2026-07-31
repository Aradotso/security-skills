---
name: ikb42603-cloud-security-essentials
description: AWS cloud security lab exercises covering IAM, encryption, network security, monitoring, and incident detection for IKB42603 course
triggers:
  - "help me with AWS cloud security labs"
  - "set up IAM users and policies"
  - "configure AWS encryption and KMS"
  - "implement VPC security groups"
  - "set up CloudTrail and CloudWatch monitoring"
  - "complete cloud security essentials lab"
  - "configure AWS security best practices"
  - "troubleshoot AWS security configurations"
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides expertise in completing AWS cloud security laboratory exercises from the IKB42603 - Cloud Computing Security Essentials course. It covers five core labs: Account Security & IAM, Secure Isolation & Multitenancy, Encryption & Key Management, Access Control & Network Security, and Monitoring/Logging/Incident Detection.

## What This Project Covers

The repository contains hands-on AWS security labs that teach:

- **Lab 1**: AWS account security, IAM users, groups, roles, policies, and MFA
- **Lab 2**: VPC isolation, multi-tenancy, and resource segregation
- **Lab 3**: Data encryption at rest and in transit, AWS KMS usage
- **Lab 4**: Security groups, NACLs, VPC peering, and access control
- **Lab 5**: CloudTrail logging, CloudWatch monitoring, and incident detection

## Repository Setup

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create lab structure
mkdir -p Lab{1..5}
touch Lab0_Environment_Setup.md
touch Lab{1..5}_*.md
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
└── Lab5_Monitoring_Logging_and_Incident_Detection.md
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-1

# Create access keys for programmatic access
aws iam create-access-key --user-name lab-user-1

# Attach a managed policy
aws iam attach-user-policy \
  --user-name lab-user-1 \
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
        "arn:aws:s3:::my-secure-bucket",
        "arn:aws:s3:::my-secure-bucket/*"
      ]
    }
  ]
}
```

```bash
# Create the policy
aws iam create-policy \
  --policy-name S3ReadOnlyCustom \
  --policy-document file://s3-readonly-policy.json

# Attach to user
aws iam attach-user-policy \
  --user-name lab-user-1 \
  --policy-arn arn:aws:iam::${AWS_ACCOUNT_ID}:policy/S3ReadOnlyCustom
```

### Creating IAM Groups

```bash
# Create group
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

### Enabling MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-1-mfa \
  --outfile /tmp/QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA (requires two consecutive TOTP codes)
aws iam enable-mfa-device \
  --user-name lab-user-1 \
  --serial-number arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/lab-user-1-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### Creating IAM Roles

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

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Access-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Instance-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Instance-Profile \
  --role-name EC2-S3-Access-Role
```

## Lab 2: Secure Isolation and Multitenancy

### Creating Isolated VPCs

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab2-VPC}]'

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

# Create and attach internet gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab2-IGW}]'

aws ec2 attach-internet-gateway \
  --vpc-id vpc-xxxxx \
  --internet-gateway-id igw-xxxxx
```

### Resource Tagging for Multi-Tenancy

```bash
# Tag resources by tenant
aws ec2 create-tags \
  --resources i-xxxxx \
  --tags Key=Tenant,Value=TenantA Key=Environment,Value=Production

# List resources by tenant
aws ec2 describe-instances \
  --filters "Name=tag:Tenant,Values=TenantA"
```

### VPC Endpoints for Secure Access

```bash
# Create S3 VPC endpoint (gateway)
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxxxx

# Create interface endpoint for other services
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-east-1.ec2 \
  --subnet-ids subnet-xxxxx
```

## Lab 3: Encryption and Key Management

### Creating and Using KMS Keys

```bash
# Create customer managed key
aws kms create-key \
  --description "Lab3 encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create alias
aws kms create-alias \
  --alias-name alias/lab3-key \
  --target-key-id xxxxx-xxxx-xxxx-xxxx-xxxxx

# Encrypt data
echo "sensitive data" > plaintext.txt
aws kms encrypt \
  --key-id alias/lab3-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode
```

### S3 Bucket Encryption

```bash
# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket my-secure-bucket \
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
aws s3 cp sensitive-file.txt s3://my-secure-bucket/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id alias/lab3-key
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/lab3-key \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=Encrypted-Volume}]'

# Enable encryption by default
aws ec2 enable-ebs-encryption-by-default
```

### RDS Encryption

```bash
# Create encrypted RDS instance
aws rds create-db-instance \
  --db-instance-identifier lab3-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password "${DB_PASSWORD}" \
  --allocated-storage 20 \
  --storage-encrypted \
  --kms-key-id alias/lab3-key
```

## Lab 4: Access Control and Network Security

### Security Groups

```bash
# Create security group
aws ec2 create-security-group \
  --group-name WebServerSG \
  --description "Security group for web servers" \
  --vpc-id vpc-xxxxx

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP only
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24

# Allow traffic from another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-backend \
  --protocol tcp \
  --port 3306 \
  --source-group sg-frontend
```

### Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-xxxxx \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=CustomNACL}]'

# Add inbound rule (allow HTTP)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Add outbound rule (allow all)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --egress \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Associate with subnet
aws ec2 replace-network-acl-association \
  --association-id aclassoc-xxxxx \
  --network-acl-id acl-xxxxx
```

### VPC Flow Logs

```bash
# Create flow log to CloudWatch
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::${AWS_ACCOUNT_ID}:role/flowlogsRole

# Create flow log to S3
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::vpc-flow-logs-bucket
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --region us-east-1

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
  --policy file://bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name lab5-trail \
  --s3-bucket-name cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name lab5-trail

# Enable log file validation
aws cloudtrail update-trail \
  --name lab5-trail \
  --enable-log-file-validation
```

### CloudWatch Alarms

```bash
# Create SNS topic for alarms
aws sns create-topic --name SecurityAlerts

# Subscribe email to topic
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:${AWS_ACCOUNT_ID}:SecurityAlerts \
  --protocol email \
  --notification-endpoint ${ADMIN_EMAIL}

# Create metric alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICallsCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:${AWS_ACCOUNT_ID}:SecurityAlerts

# Create alarm for root account usage
aws cloudwatch put-metric-alarm \
  --alarm-name RootAccountUsage \
  --alarm-description "Alert when root account is used" \
  --metric-name RootAccountUsageCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:${AWS_ACCOUNT_ID}:SecurityAlerts
```

### CloudWatch Logs Insights Queries

```bash
# Query failed login attempts
aws logs start-query \
  --log-group-name /aws/cloudtrail \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, userIdentity.principalId, errorCode
    | filter errorCode = "UnauthorizedOperation" or errorCode = "AccessDenied"
    | sort @timestamp desc
    | limit 20'

# Query S3 bucket access
aws logs start-query \
  --log-group-name /aws/cloudtrail \
  --start-time $(date -d '24 hours ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, userIdentity.principalId, eventName, requestParameters.bucketName
    | filter eventSource = "s3.amazonaws.com"
    | stats count() by eventName
    | sort count desc'
```

### GuardDuty Integration

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

# List findings
aws guardduty list-findings \
  --detector-id xxxxx \
  --finding-criteria '{"Criterion":{"severity":{"Gte":7}}}'

# Get finding details
aws guardduty get-findings \
  --detector-id xxxxx \
  --finding-ids xxxxx
```

### AWS Config Rules

```bash
# Enable AWS Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::${AWS_ACCOUNT_ID}:role/config-role

aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=config-bucket-${AWS_ACCOUNT_ID}

aws configservice start-configuration-recorder \
  --configuration-recorder-name default

# Deploy managed rule (encrypted volumes)
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "encrypted-volumes",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "ENCRYPTED_VOLUMES"
    }
  }'

# Deploy managed rule (S3 bucket public read prohibited)
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "s3-bucket-public-read-prohibited",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "S3_BUCKET_PUBLIC_READ_PROHIBITED"
    }
  }'
```

## Common Lab Patterns

### Lab Documentation Template

```markdown
# Lab X: [Lab Title]

## Objective
[What you're trying to achieve]

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Environment
- AWS Region: us-east-1
- Services Used: IAM, EC2, S3, etc.
- Tools: AWS CLI, AWS Console

## Step-by-Step Implementation

### Step 1: [Task Name]
[Description]

**Commands:**
```bash
# Command with explanation
aws service action --parameter value
```

**Screenshot:**
![Step 1 Screenshot](screenshots/step1.png)

### Step 2: [Task Name]
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

### Git Workflow

```bash
# Check status
git status

# Add specific lab files
git add Lab1_Account_Security_and_IAM.md
git add screenshots/

# Or add all changes
git add .

# Commit with descriptive message
git commit -m "Complete Lab 1: IAM users, groups, and MFA configuration"

# Push to GitHub
git push origin main

# Create feature branch for work in progress
git checkout -b lab2-wip
git push -u origin lab2-wip
```

## Security Best Practices

### Credential Management

```bash
# Configure AWS CLI with credentials
aws configure
# AWS Access Key ID: (stored in ~/.aws/credentials)
# AWS Secret Access Key: (stored in ~/.aws/credentials)
# Default region: us-east-1
# Default output format: json

# Use environment variables (preferred for labs)
export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"
export AWS_DEFAULT_REGION="us-east-1"

# Use named profiles
aws configure --profile lab-profile
aws s3 ls --profile lab-profile
```

### Least Privilege IAM Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeSecurityGroups",
        "ec2:CreateSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

### Resource Cleanup

```bash
# Delete IAM user
aws iam delete-access-key --user-name lab-user-1 --access-key-id AKIA...
aws iam detach-user-policy --user-name lab-user-1 --policy-arn arn:aws:iam::...
aws iam delete-user --user-name lab-user-1

# Delete security group
aws ec2 delete-security-group --group-id sg-xxxxx

# Delete VPC (requires deleting dependencies first)
aws ec2 delete-subnet --subnet-id subnet-xxxxx
aws ec2 detach-internet-gateway --vpc-id vpc-xxxxx --internet-gateway-id igw-xxxxx
aws ec2 delete-internet-gateway --internet-gateway-id igw-xxxxx
aws ec2 delete-vpc --vpc-id vpc-xxxxx

# Terminate EC2 instances
aws ec2 terminate-instances --instance-ids i-xxxxx

# Delete S3 bucket (must be empty)
aws s3 rm s3://my-bucket --recursive
aws s3api delete-bucket --bucket my-bucket
```

## Troubleshooting

### Permission Errors

```bash
# Check who you're authenticated as
aws sts get-caller-identity

# Simulate IAM policy
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::${AWS_ACCOUNT_ID}:user/lab-user-1 \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt

# Decode authorization failure message
aws sts decode-authorization-message \
  --encoded-message <encoded-message-from-error>
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-xxxxx"

# Check security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Check NACL rules
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=vpc-xxxxx"

# Test reachability (VPC Reachability Analyzer)
aws ec2 create-network-insights-path \
  --source i-xxxxx \
  --destination i-yyyyy \
  --protocol tcp \
  --destination-port 22

aws ec2 start-network-insights-analysis \
  --network-insights-path-id nip-xxxxx
```

### CloudTrail Not Logging

```bash
# Check trail status
aws cloudtrail get-trail-status --name lab5-trail

# Verify S3 bucket permissions
aws s3api get-bucket-policy --bucket cloudtrail-logs-${AWS_ACCOUNT_ID}

# Check if logging is enabled
aws cloudtrail describe-trails --trail-name-list lab5-trail

# Validate log file integrity
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:us-east-1:${AWS_ACCOUNT_ID}:trail/lab5-trail \
  --start-time 2026-07-27T00:00:00Z
```

### KMS Key Access Issues

```bash
# Check key policy
aws kms get-key-policy \
  --key-id alias/lab3-key \
  --policy-name default

# List grants on key
aws kms list-grants --key-id alias/lab3-key

# Test encryption permissions
aws kms encrypt \
  --key-id alias/lab3-key \
  --plaintext "test" \
  --query CiphertextBlob \
  --output text
```

## Assessment Checklist

Before submitting each lab:

- [ ] All tasks completed as per lab instructions
- [ ] Commands documented with explanations
- [ ] Screenshots included for key steps
- [ ] README.md follows template structure
- [ ] Code blocks properly formatted
- [ ] No hardcoded credentials or secrets
- [ ] Resources tagged appropriately
- [ ] Cleanup steps documented
- [ ] Challenges and solutions noted
- [ ] References cited
- [ ] Git commits are descriptive and atomic
- [ ] Repository pushed to GitHub

## Additional Resources

- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [AWS Security Best Practices](https://docs.aws.amazon.com/security/)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
