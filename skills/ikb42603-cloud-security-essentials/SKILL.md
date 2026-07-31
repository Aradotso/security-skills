---
name: ikb42603-cloud-security-essentials
description: Educational AWS cloud security lab exercises covering IAM, encryption, VPC isolation, network security, and monitoring with CloudTrail and CloudWatch
triggers:
  - how do I complete the cloud security labs
  - help me with AWS security essentials lab
  - show me IAM security best practices
  - configure AWS encryption and key management
  - set up VPC isolation and security groups
  - implement CloudTrail and CloudWatch monitoring
  - complete IKB42603 lab exercises
  - AWS cloud security fundamentals practice
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603 Cloud Computing Security Essentials is an educational repository containing five comprehensive AWS security laboratory exercises. The labs cover fundamental cloud security concepts including IAM (Identity and Access Management), secure multi-tenancy, encryption and key management, network security with VPCs and Security Groups, and monitoring/logging with CloudTrail and CloudWatch.

This skill helps AI agents guide students through practical AWS security implementations following academic best practices.

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

## Initial Setup

### Clone and Initialize Repository

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
git commit -m "Initial lab setup"
git push origin main
```

### AWS CLI Configuration

```bash
# Install AWS CLI (if not installed)
pip install awscli

# Configure AWS credentials
aws configure
# AWS Access Key ID: ${AWS_ACCESS_KEY_ID}
# AWS Secret Access Key: ${AWS_SECRET_ACCESS_KEY}
# Default region: us-east-1
# Default output format: json

# Verify configuration
aws sts get-caller-identity
```

## Lab 1: Account Security and IAM

### Creating IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-admin

# Create access key for programmatic access
aws iam create-access-key --user-name lab-user-admin

# Attach administrative policy
aws iam attach-user-policy \
  --user-name lab-user-admin \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

### IAM Groups and Permissions

```bash
# Create IAM group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-admin \
  --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
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
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::lab-bucket-${AWS_ACCOUNT_ID}/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::lab-bucket-${AWS_ACCOUNT_ID}"
    }
  ]
}
```

```bash
# Create custom policy
aws iam create-policy \
  --policy-name S3ReadWritePolicy \
  --policy-document file://s3-policy.json

# Attach custom policy
aws iam attach-user-policy \
  --user-name lab-user-admin \
  --policy-arn arn:aws:iam::${AWS_ACCOUNT_ID}:policy/S3ReadWritePolicy
```

### Enable MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-mfa \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA for user (requires authentication codes)
aws iam enable-mfa-device \
  --user-name lab-user-admin \
  --serial-number arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/lab-user-mfa \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

## Lab 2: Secure Isolation and Multitenancy

### Create VPC with Isolated Subnets

```bash
# Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab-VPC}]' \
  --query 'Vpc.VpcId' \
  --output text)

# Create public subnet
PUBLIC_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-Subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# Create private subnet
PRIVATE_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Private-Subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# Create Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab-IGW}]' \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
```

### Configure Route Tables

```bash
# Create route table for public subnet
PUBLIC_RT=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public-RT}]' \
  --query 'RouteTable.RouteTableId' \
  --output text)

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id $PUBLIC_RT \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associate route table with public subnet
aws ec2 associate-route-table \
  --subnet-id $PUBLIC_SUBNET \
  --route-table-id $PUBLIC_RT
```

### Network ACLs for Isolation

```bash
# Create Network ACL
NACL_ID=$(aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Private-NACL}]' \
  --query 'NetworkAcl.NetworkAclId' \
  --output text)

# Add inbound rule (allow SSH from specific IP)
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=22,To=22 \
  --cidr-block 10.0.1.0/24 \
  --rule-action allow \
  --ingress

# Add outbound rule
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol -1 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow \
  --egress

# Associate NACL with private subnet
aws ec2 replace-network-acl-association \
  --association-id $(aws ec2 describe-network-acls \
    --filters "Name=association.subnet-id,Values=$PRIVATE_SUBNET" \
    --query 'NetworkAcls[0].Associations[0].NetworkAclAssociationId' \
    --output text) \
  --network-acl-id $NACL_ID
```

## Lab 3: Encryption and Key Management

### Create KMS Key

```bash
# Create KMS key
KEY_ID=$(aws kms create-key \
  --description "Lab encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --query 'KeyMetadata.KeyId' \
  --output text)

# Create alias for key
aws kms create-alias \
  --alias-name alias/lab-key \
  --target-key-id $KEY_ID

# List KMS keys
aws kms list-keys
```

### Encrypt and Decrypt Data

```bash
# Encrypt plaintext
echo "Sensitive data" > plaintext.txt

aws kms encrypt \
  --key-id alias/lab-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.dat

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.dat \
  --output text \
  --query Plaintext | base64 --decode
```

### S3 Bucket Encryption

```bash
# Create S3 bucket with encryption
aws s3api create-bucket \
  --bucket lab-secure-bucket-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket lab-secure-bucket-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab-key"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Upload encrypted object
aws s3 cp plaintext.txt s3://lab-secure-bucket-${AWS_ACCOUNT_ID}/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id alias/lab-key
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
VOLUME_ID=$(aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/lab-key \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=Encrypted-Volume}]' \
  --query 'VolumeId' \
  --output text)

# Verify encryption
aws ec2 describe-volumes \
  --volume-ids $VOLUME_ID \
  --query 'Volumes[0].[Encrypted,KmsKeyId]'
```

## Lab 4: Access Control and Network Security

### Security Groups Configuration

```bash
# Create security group for web server
WEB_SG=$(aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Security group for web servers" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG \
  --protocol tcp \
  --port 22 \
  --cidr ${MY_IP}/32
```

### Database Security Group

```bash
# Create security group for database
DB_SG=$(aws ec2 create-security-group \
  --group-name database-sg \
  --description "Security group for database servers" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

# Allow MySQL only from web server security group
aws ec2 authorize-security-group-ingress \
  --group-id $DB_SG \
  --protocol tcp \
  --port 3306 \
  --source-group $WEB_SG
```

### Launch EC2 Instance with Security

```bash
# Launch instance in public subnet
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name ${KEY_PAIR_NAME} \
  --security-group-ids $WEB_SG \
  --subnet-id $PUBLIC_SUBNET \
  --associate-public-ip-address \
  --iam-instance-profile Name=EC2-CloudWatch-Role \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web-Server}]' \
  --query 'Instances[0].InstanceId' \
  --output text)

# Get instance public IP
aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Enable CloudTrail

```bash
# Create S3 bucket for CloudTrail logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Apply bucket policy for CloudTrail
cat > cloudtrail-policy.json <<EOF
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
  --policy file://cloudtrail-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name lab-trail \
  --s3-bucket-name cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name lab-trail
```

### CloudWatch Alarms

```bash
# Create SNS topic for alerts
TOPIC_ARN=$(aws sns create-topic \
  --name security-alerts \
  --query 'TopicArn' \
  --output text)

# Subscribe email to topic
aws sns subscribe \
  --topic-arn $TOPIC_ARN \
  --protocol email \
  --notification-endpoint ${YOUR_EMAIL}

# Create alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name unauthorized-api-calls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICalls \
  --namespace AWS/CloudTrail \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions $TOPIC_ARN
```

### CloudWatch Logs for EC2

```bash
# Create log group
aws logs create-log-group \
  --log-group-name /aws/ec2/web-server

# Set retention policy
aws logs put-retention-policy \
  --log-group-name /aws/ec2/web-server \
  --retention-in-days 7

# Create metric filter for failed SSH attempts
aws logs put-metric-filter \
  --log-group-name /aws/ec2/web-server \
  --filter-name failed-ssh-attempts \
  --filter-pattern "[Mon, day, timestamp, ip, id, status = *failed*]" \
  --metric-transformations \
    metricName=FailedSSHAttempts,metricNamespace=Security,metricValue=1
```

### Query CloudTrail Logs

```bash
# Lookup recent events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin \
  --max-results 10

# Query specific user activity
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=lab-user-admin \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --query 'Events[*].[EventTime,EventName,Username]' \
  --output table
```

### CloudWatch Insights Queries

```bash
# Query failed authentication attempts
aws logs start-query \
  --log-group-name /aws/cloudtrail \
  --start-time $(date -u -d '24 hours ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, errorCode, errorMessage, userIdentity.principalId
| filter errorCode like /Unauthorized|Forbidden|AccessDenied/
| sort @timestamp desc
| limit 20'
```

## Documentation Best Practices

### Lab Documentation Template

```markdown
# Lab X: [Lab Title]

## Objective
[Clear statement of what students will learn]

## Learning Outcomes
- Outcome 1
- Outcome 2
- Outcome 3

## Prerequisites
- AWS Account
- AWS CLI configured
- Basic understanding of [relevant concept]

## Environment
- AWS Region: us-east-1
- Tools: AWS CLI, AWS Console

## Implementation Steps

### Step 1: [Task Name]

**Command:**
```bash
aws service command --parameter value
```

**Screenshot:**
![Description](screenshots/lab-x-step-1.png)

**Explanation:**
[Detailed explanation of what this step accomplishes]

### Step 2: [Next Task]
[Continue pattern...]

## Verification
[How to verify the lab was completed successfully]

## Challenges Encountered
- Challenge 1 and resolution
- Challenge 2 and resolution

## Lessons Learned
[Key takeaways from the lab]

## References
- [AWS Documentation Link]
- [Related Tutorial]

## Cleanup
```bash
# Commands to remove resources
aws service delete-resource --id $RESOURCE_ID
```
```

## Common Patterns

### Resource Tagging Strategy

```bash
# Apply consistent tags to all resources
TAGS='ResourceType=*,Tags=[{Key=Project,Value=IKB42603},{Key=Lab,Value=Lab1},{Key=Environment,Value=Learning}]'

# Tag existing resource
aws ec2 create-tags \
  --resources $INSTANCE_ID \
  --tags Key=Project,Value=IKB42603 Key=Lab,Value=Lab1
```

### Git Workflow for Lab Submissions

```bash
# Create feature branch for each lab
git checkout -b lab1-iam-security

# Work on lab, commit frequently
git add Lab1_Account_Security_and_IAM.md screenshots/
git commit -m "Complete IAM user creation with MFA"

# Push to GitHub
git push origin lab1-iam-security

# Merge to main when complete
git checkout main
git merge lab1-iam-security
git push origin main
```

### Screenshot Organization

```bash
# Create screenshots directory
mkdir -p screenshots/lab{1..5}

# Naming convention
# screenshots/lab1/01-iam-user-creation.png
# screenshots/lab1/02-mfa-enabled.png
# screenshots/lab2/01-vpc-creation.png
```

## Troubleshooting

### AWS CLI Authentication Issues

```bash
# Verify credentials are configured
aws sts get-caller-identity

# Check region configuration
aws configure get region

# Test with simple command
aws s3 ls

# Re-configure if needed
aws configure
```

### Permission Denied Errors

```bash
# Check IAM permissions for current user
aws iam get-user

# List attached policies
aws iam list-attached-user-policies --user-name $(aws iam get-user --query 'User.UserName' --output text)

# Verify policy details
aws iam get-policy-version \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --version-id v1
```

### CloudTrail Not Logging

```bash
# Verify trail status
aws cloudtrail get-trail-status --name lab-trail

# Check if logging is enabled
aws cloudtrail describe-trails --trail-name-list lab-trail

# Restart logging
aws cloudtrail start-logging --name lab-trail
```

### VPC Connectivity Issues

```bash
# Verify route table associations
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID"

# Check security group rules
aws ec2 describe-security-groups --group-ids $WEB_SG

# Verify network ACLs
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=$VPC_ID"

# Test connectivity from instance
aws ec2-instance-connect send-ssh-public-key \
  --instance-id $INSTANCE_ID \
  --availability-zone us-east-1a \
  --instance-os-user ec2-user \
  --ssh-public-key file://~/.ssh/id_rsa.pub
```

### KMS Key Access Issues

```bash
# List KMS keys
aws kms list-keys

# Get key policy
aws kms get-key-policy \
  --key-id alias/lab-key \
  --policy-name default

# Grant user permission to use key
aws kms create-grant \
  --key-id alias/lab-key \
  --grantee-principal arn:aws:iam::${AWS_ACCOUNT_ID}:user/lab-user-admin \
  --operations Encrypt Decrypt
```

## Resource Cleanup

### Complete Cleanup Script

```bash
#!/bin/bash
# cleanup.sh - Remove all lab resources

# Terminate EC2 instances
aws ec2 terminate-instances --instance-ids $INSTANCE_ID

# Delete security groups
aws ec2 delete-security-group --group-id $WEB_SG
aws ec2 delete-security-group --group-id $DB_SG

# Delete subnets
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET

# Detach and delete Internet Gateway
aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID

# Delete VPC
aws ec2 delete-vpc --vpc-id $VPC_ID

# Delete KMS key (schedule deletion)
aws kms schedule-key-deletion --key-id alias/lab-key --pending-window-in-days 7

# Empty and delete S3 buckets
aws s3 rm s3://lab-secure-bucket-${AWS_ACCOUNT_ID} --recursive
aws s3api delete-bucket --bucket lab-secure-bucket-${AWS_ACCOUNT_ID}

aws s3 rm s3://cloudtrail-logs-${AWS_ACCOUNT_ID} --recursive
aws s3api delete-bucket --bucket cloudtrail-logs-${AWS_ACCOUNT_ID}

# Delete CloudTrail
aws cloudtrail delete-trail --name lab-trail

# Delete CloudWatch log groups
aws logs delete-log-group --log-group-name /aws/ec2/web-server

# Delete IAM resources
aws iam detach-user-policy --user-name lab-user-admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam delete-user --user-name lab-user-admin

echo "Cleanup complete"
```

## Academic Integrity Notes

When assisting students:
- Encourage understanding over copying
- Explain the "why" behind each configuration
- Ask questions to verify comprehension
- Suggest improvements to documentation
- Emphasize security best practices
- Remind about proper attribution of sources
