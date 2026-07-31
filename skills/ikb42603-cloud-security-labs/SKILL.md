---
name: ikb42603-cloud-security-labs
description: AWS cloud security lab exercises covering IAM, encryption, network security, monitoring, and incident detection for educational purposes
triggers:
  - "help me with cloud security labs"
  - "setup AWS security lab environment"
  - "configure IAM users and policies"
  - "implement AWS encryption and KMS"
  - "setup VPC security groups and network isolation"
  - "configure CloudTrail and CloudWatch monitoring"
  - "complete cloud computing security assignment"
  - "AWS security best practices lab"
---

# IKB42603 Cloud Security Labs Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides guidance for completing AWS cloud security laboratory exercises covering account security, IAM, secure isolation, encryption, access control, network security, and monitoring. The labs are designed for educational purposes to teach cloud computing security essentials.

## What This Project Covers

The IKB42603 lab series includes five core security domains:

- **Lab 1**: Account Security and IAM (Identity and Access Management)
- **Lab 2**: Secure Isolation and Multitenancy (VPC, Subnets, Security Groups)
- **Lab 3**: Encryption and Key Management (AWS KMS, S3 encryption)
- **Lab 4**: Access Control and Network Security (NACLs, Security Groups, Bastion Hosts)
- **Lab 5**: Monitoring, Logging, and Incident Detection (CloudTrail, CloudWatch)

## Repository Setup

### Initial Project Structure

```bash
# Clone your repository
git clone https://github.com/<your-username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS

# Create lab documentation files
touch README.md
touch Lab0_Environment_Setup.md
touch Lab1_Account_Security_and_IAM.md
touch Lab2_Secure_Isolation_and_Multitenancy.md
touch Lab3_Encryption_and_Key_Management.md
touch Lab4_Access_Control_and_Network_Security.md
touch Lab5_Monitoring_Logging_and_Incident_Detection.md

# Create directories for each lab
mkdir -p Lab1_Account_Security_and_IAM/{policies,scripts,screenshots}
mkdir -p Lab2_Secure_Isolation_and_Multitenancy/{configs,scripts,screenshots}
mkdir -p Lab3_Encryption_and_Key_Management/{scripts,screenshots}
mkdir -p Lab4_Access_Control_and_Network_Security/{configs,scripts,screenshots}
mkdir -p Lab5_Monitoring_Logging_and_Incident_Detection/{queries,scripts,screenshots}
```

### Git Workflow

```bash
# Check status
git status

# Stage changes
git add .

# Commit with meaningful message
git commit -m "Complete Lab 1: IAM configuration and MFA setup"

# Push to GitHub
git push origin main
```

## Lab 1: Account Security and IAM

### AWS CLI Setup

```bash
# Install AWS CLI (Ubuntu/Debian)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS credentials
aws configure
# AWS Access Key ID: [from env var AWS_ACCESS_KEY_ID]
# AWS Secret Access Key: [from env var AWS_SECRET_ACCESS_KEY]
# Default region: us-east-1
# Default output format: json
```

### Create IAM Users

```bash
# Create a new IAM user
aws iam create-user --user-name lab-user-1

# Create a login profile (console password)
aws iam create-login-profile \
  --user-name lab-user-1 \
  --password "TempPassword123!" \
  --password-reset-required

# List all IAM users
aws iam list-users
```

### IAM Policy Examples

**Read-Only S3 Policy** (`Lab1_Account_Security_and_IAM/policies/s3-readonly.json`):

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
        "arn:aws:s3:::my-lab-bucket",
        "arn:aws:s3:::my-lab-bucket/*"
      ]
    }
  ]
}
```

**EC2 Limited Access Policy** (`Lab1_Account_Security_and_IAM/policies/ec2-limited.json`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeKeyPairs"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "Lab"
        }
      }
    }
  ]
}
```

### Attach Policies

```bash
# Attach AWS managed policy
aws iam attach-user-policy \
  --user-name lab-user-1 \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Create and attach custom policy
aws iam create-policy \
  --policy-name S3ReadOnlyCustom \
  --policy-document file://Lab1_Account_Security_and_IAM/policies/s3-readonly.json

# Get your account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Attach custom policy
aws iam attach-user-policy \
  --user-name lab-user-1 \
  --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/S3ReadOnlyCustom
```

### Enable MFA

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name lab-user-1-mfa \
  --outfile Lab1_Account_Security_and_IAM/mfa-qr-code.png \
  --bootstrap-method QRCodePNG

# Enable MFA device (requires two consecutive MFA codes)
aws iam enable-mfa-device \
  --user-name lab-user-1 \
  --serial-number arn:aws:iam::${ACCOUNT_ID}:mfa/lab-user-1-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### IAM Groups and Roles

```bash
# Create IAM group
aws iam create-group --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Add user to group
aws iam add-user-to-group \
  --user-name lab-user-1 \
  --group-name Developers

# Create IAM role for EC2
aws iam create-role \
  --role-name EC2-S3-Access-Role \
  --assume-role-policy-document file://Lab1_Account_Security_and_IAM/policies/ec2-trust-policy.json
```

**EC2 Trust Policy** (`Lab1_Account_Security_and_IAM/policies/ec2-trust-policy.json`):

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

## Lab 2: Secure Isolation and Multitenancy

### VPC Creation

```bash
# Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab-VPC}]' \
  --query 'Vpc.VpcId' \
  --output text)

echo "VPC ID: ${VPC_ID}"

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id ${VPC_ID} \
  --enable-dns-hostnames
```

### Subnet Configuration

```bash
# Create public subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Lab-Public-Subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# Create private subnet
PRIVATE_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Lab-Private-Subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# Enable auto-assign public IP for public subnet
aws ec2 modify-subnet-attribute \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --map-public-ip-on-launch
```

### Internet Gateway and Route Tables

```bash
# Create Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab-IGW}]' \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --vpc-id ${VPC_ID} \
  --internet-gateway-id ${IGW_ID}

# Create public route table
PUBLIC_RT_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Lab-Public-RT}]' \
  --query 'RouteTable.RouteTableId' \
  --output text)

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id ${PUBLIC_RT_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${IGW_ID}

# Associate public subnet with route table
aws ec2 associate-route-table \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --route-table-id ${PUBLIC_RT_ID}
```

### Security Groups

```bash
# Create web server security group
WEB_SG_ID=$(aws ec2 create-security-group \
  --group-name lab-web-sg \
  --description "Security group for web servers" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id ${WEB_SG_ID} \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id ${WEB_SG_ID} \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id ${WEB_SG_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/32

# Create database security group
DB_SG_ID=$(aws ec2 create-security-group \
  --group-name lab-db-sg \
  --description "Security group for database servers" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow MySQL only from web security group
aws ec2 authorize-security-group-ingress \
  --group-id ${DB_SG_ID} \
  --protocol tcp \
  --port 3306 \
  --source-group ${WEB_SG_ID}
```

### Network ACLs

```bash
# Create network ACL
NACL_ID=$(aws ec2 create-network-acl \
  --vpc-id ${VPC_ID} \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Lab-NACL}]' \
  --query 'NetworkAcl.NetworkAclId' \
  --output text)

# Allow inbound HTTP
aws ec2 create-network-acl-entry \
  --network-acl-id ${NACL_ID} \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow \
  --ingress

# Allow outbound HTTP responses
aws ec2 create-network-acl-entry \
  --network-acl-id ${NACL_ID} \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow \
  --egress

# Associate NACL with subnet
aws ec2 replace-network-acl-association \
  --association-id $(aws ec2 describe-network-acls \
    --filters "Name=association.subnet-id,Values=${PUBLIC_SUBNET_ID}" \
    --query 'NetworkAcls[0].Associations[0].NetworkAclAssociationId' \
    --output text) \
  --network-acl-id ${NACL_ID}
```

## Lab 3: Encryption and Key Management

### KMS Key Creation

```bash
# Create KMS key
KEY_ID=$(aws kms create-key \
  --description "Lab encryption key for S3" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS \
  --query 'KeyMetadata.KeyId' \
  --output text)

# Create alias for the key
aws kms create-alias \
  --alias-name alias/lab-s3-key \
  --target-key-id ${KEY_ID}

# Describe key
aws kms describe-key --key-id ${KEY_ID}
```

### S3 Bucket Encryption

```bash
# Create S3 bucket
BUCKET_NAME="lab-encrypted-bucket-$(date +%s)"
aws s3 mb s3://${BUCKET_NAME}

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket ${BUCKET_NAME} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "'${KEY_ID}'"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Upload encrypted file
echo "Sensitive data" > Lab3_Encryption_and_Key_Management/test-file.txt
aws s3 cp Lab3_Encryption_and_Key_Management/test-file.txt \
  s3://${BUCKET_NAME}/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id ${KEY_ID}

# Verify encryption
aws s3api head-object \
  --bucket ${BUCKET_NAME} \
  --key test-file.txt \
  --query 'ServerSideEncryption'
```

### KMS Key Policy

**Key Policy** (`Lab3_Encryption_and_Key_Management/kms-key-policy.json`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow S3 to use the key",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow specific IAM users",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:user/lab-user-1"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    }
  ]
}
```

### Encrypt/Decrypt Data

```bash
# Encrypt plaintext
echo "Secret message" > Lab3_Encryption_and_Key_Management/plaintext.txt
aws kms encrypt \
  --key-id ${KEY_ID} \
  --plaintext fileb://Lab3_Encryption_and_Key_Management/plaintext.txt \
  --query CiphertextBlob \
  --output text | base64 --decode > Lab3_Encryption_and_Key_Management/encrypted.bin

# Decrypt ciphertext
aws kms decrypt \
  --ciphertext-blob fileb://Lab3_Encryption_and_Key_Management/encrypted.bin \
  --query Plaintext \
  --output text | base64 --decode
```

### EBS Volume Encryption

```bash
# Create encrypted EBS volume
VOLUME_ID=$(aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id ${KEY_ID} \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=Lab-Encrypted-Volume}]' \
  --query 'VolumeId' \
  --output text)

# Verify encryption
aws ec2 describe-volumes \
  --volume-ids ${VOLUME_ID} \
  --query 'Volumes[0].[Encrypted,KmsKeyId]'
```

## Lab 4: Access Control and Network Security

### Bastion Host Setup

```bash
# Create key pair
aws ec2 create-key-pair \
  --key-name lab-bastion-key \
  --query 'KeyMaterial' \
  --output text > Lab4_Access_Control_and_Network_Security/lab-bastion-key.pem

chmod 400 Lab4_Access_Control_and_Network_Security/lab-bastion-key.pem

# Create bastion security group
BASTION_SG_ID=$(aws ec2 create-security-group \
  --group-name lab-bastion-sg \
  --description "Bastion host security group" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id ${BASTION_SG_ID} \
  --protocol tcp \
  --port 22 \
  --cidr ${YOUR_IP}/32

# Launch bastion host
BASTION_ID=$(aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name lab-bastion-key \
  --security-group-ids ${BASTION_SG_ID} \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Lab-Bastion}]' \
  --query 'Instances[0].InstanceId' \
  --output text)
```

### VPC Peering

```bash
# Create second VPC
VPC2_ID=$(aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Lab-VPC-2}]' \
  --query 'Vpc.VpcId' \
  --output text)

# Create peering connection
PEERING_ID=$(aws ec2 create-vpc-peering-connection \
  --vpc-id ${VPC_ID} \
  --peer-vpc-id ${VPC2_ID} \
  --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=Lab-Peering}]' \
  --query 'VpcPeeringConnection.VpcPeeringConnectionId' \
  --output text)

# Accept peering connection
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id ${PEERING_ID}

# Update route tables for peering
aws ec2 create-route \
  --route-table-id ${PUBLIC_RT_ID} \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id ${PEERING_ID}
```

### VPC Flow Logs

```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name /aws/vpc/flowlogs

# Create IAM role for VPC Flow Logs
aws iam create-role \
  --role-name VPCFlowLogsRole \
  --assume-role-policy-document file://Lab4_Access_Control_and_Network_Security/vpc-flow-logs-trust.json

# Attach policy
aws iam attach-role-policy \
  --role-name VPCFlowLogsRole \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids ${VPC_ID} \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::${ACCOUNT_ID}:role/VPCFlowLogsRole
```

### Security Group Analysis Script

**Python script** (`Lab4_Access_Control_and_Network_Security/scripts/analyze_security_groups.py`):

```python
#!/usr/bin/env python3
import boto3
import json

def analyze_security_groups():
    ec2 = boto3.client('ec2')
    
    # Get all security groups
    response = ec2.describe_security_groups()
    
    findings = []
    
    for sg in response['SecurityGroups']:
        sg_id = sg['GroupId']
        sg_name = sg['GroupName']
        
        # Check for overly permissive rules
        for rule in sg.get('IpPermissions', []):
            for ip_range in rule.get('IpRanges', []):
                if ip_range.get('CidrIp') == '0.0.0.0/0':
                    port = rule.get('FromPort', 'All')
                    findings.append({
                        'SecurityGroup': sg_name,
                        'GroupId': sg_id,
                        'Issue': 'Open to internet',
                        'Port': port,
                        'Protocol': rule.get('IpProtocol', 'All')
                    })
    
    return findings

if __name__ == '__main__':
    results = analyze_security_groups()
    print(json.dumps(results, indent=2))
```

## Lab 5: Monitoring, Logging, and Incident Detection

### CloudTrail Setup

```bash
# Create S3 bucket for CloudTrail logs
TRAIL_BUCKET="lab-cloudtrail-logs-$(date +%s)"
aws s3 mb s3://${TRAIL_BUCKET}

# Apply bucket policy for CloudTrail
aws s3api put-bucket-policy \
  --bucket ${TRAIL_BUCKET} \
  --policy file://Lab5_Monitoring_Logging_and_Incident_Detection/cloudtrail-bucket-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name lab-trail \
  --s3-bucket-name ${TRAIL_BUCKET} \
  --is-multi-region-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging --name lab-trail

# Verify trail status
aws cloudtrail get-trail-status --name lab-trail
```

**CloudTrail Bucket Policy** (`Lab5_Monitoring_Logging_and_Incident_Detection/cloudtrail-bucket-policy.json`):

```json
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
      "Resource": "arn:aws:s3:::BUCKET_NAME"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::BUCKET_NAME/AWSLogs/ACCOUNT_ID/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
```

### CloudWatch Alarms

```bash
# Create SNS topic for alerts
TOPIC_ARN=$(aws sns create-topic \
  --name lab-security-alerts \
  --query 'TopicArn' \
  --output text)

# Subscribe email to topic
aws sns subscribe \
  --topic-arn ${TOPIC_ARN} \
  --protocol email \
  --notification-endpoint ${YOUR_EMAIL}

# Create metric filter for failed login attempts
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/logs \
  --filter-name FailedConsoleLogins \
  --filter-pattern '{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }' \
  --metric-transformations \
    metricName=ConsoleLoginFailures,\
metricNamespace=CloudTrailMetrics,\
metricValue=1

# Create alarm for failed logins
aws cloudwatch put-metric-alarm \
  --alarm-name lab-failed-logins \
  --alarm-description "Alert on multiple failed console logins" \
  --metric-name ConsoleLoginFailures \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions ${TOPIC_ARN}
```

### CloudWatch Insights Queries

**Failed authentication attempts** (`Lab5_Monitoring_Logging_and_Incident_Detection/queries/failed-auth.txt`):

```
fields @timestamp, userIdentity.principalId, sourceIPAddress, errorMessage
| filter eventName = "ConsoleLogin" and errorMessage = "Failed authentication"
| sort @timestamp desc
| limit 20
```

**Root account usage** (`Lab5_Monitoring_Logging_and_Incident_Detection/queries/root-usage.txt`):

```
fields @timestamp, eventName, sourceIPAddress, userAgent
| filter userIdentity.type = "Root" and userIdentity.invokedBy NOT EXISTS
| sort @timestamp desc
```

**Unauthorized API calls** (`Lab5_Monitoring_Logging_and_Incident_Detection/queries/unauthorized-calls.txt`):

```
fields @timestamp, eventName, sourceIPAddress, errorCode, errorMessage
| filter errorCode in ["AccessDenied", "UnauthorizedOperation"]
| stats count() by eventName, sourceIPAddress
| sort count desc
```

### GuardDuty Setup

```bash
# Enable GuardDuty
DETECTOR_ID=$(aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES \
  --query 'DetectorId' \
  --output text)

# List findings
aws guardduty list-findings \
  --detector-id ${DETECTOR_ID} \
  --max-results 50

# Get finding details
aws guardduty get-findings \
  --detector-id ${DETECTOR_ID} \
  --finding-ids <finding-id>
```

### Security Hub Integration

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Enable AWS Foundational Security Best Practices standard
aws securityhub batch-enable-standards \
  --standards-subscription-requests '[{
    "StandardsArn": "arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0"
  }]'

# Get security findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}' \
  --max-results 10
```

### Lambda Function for Automated Response

**Automated remediation** (`Lab5_Monitoring_Logging_and_Incident_Detection/scripts/auto_remediate.py`):

```python
#!/usr/bin/env python3
import boto3
import json

def lambda_handler(event, context):
    """
    Automatically remediate security group rules that are open to the internet
    """
    ec2 = boto3.client('ec2')
    sns = boto3.client('sns')
    
    # Parse CloudWatch event
    detail = event['detail']
    
    if detail['eventName'] == 'AuthorizeSecurityGroupIngress':
        sg_id = detail['requestParameters']['groupId']
        
        # Check if rule allows 0.0.0.0/0
        ip_permissions = detail['requestParameters']['ipPermissions']['items']
        
        for permission in ip_permissions:
            for ip_range in permission.get('ipRanges', {}).get('items', []):
                if ip_range.get('cidrIp') == '0.0.0.0/0':
                    # Revoke the rule
                    try:
                        ec2.revoke_security_
