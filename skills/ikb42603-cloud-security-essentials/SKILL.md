---
name: ikb42603-cloud-security-essentials
description: AWS cloud security lab exercises covering IAM, VPC, encryption, monitoring, and incident detection for hands-on security learning
triggers:
  - "help me with AWS cloud security labs"
  - "how do I configure IAM security in AWS"
  - "set up VPC isolation and security groups"
  - "implement AWS KMS encryption"
  - "configure CloudTrail and CloudWatch monitoring"
  - "complete cloud computing security exercises"
  - "AWS security best practices lab"
  - "hands-on AWS security configuration"
---

# IKB42603 Cloud Security Essentials Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS is an educational repository containing hands-on laboratory exercises for learning AWS cloud security fundamentals. The project covers five core security domains: IAM and account security, secure isolation and multitenancy, encryption and key management, access control and network security, and monitoring/logging/incident detection.

This skill helps developers and students complete practical AWS security implementations using the AWS Console, AWS CLI, and infrastructure-as-code approaches.

## Repository Structure

The project is organized into five lab modules:

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

## Installation & Setup

### Clone the Repository

```bash
git clone https://github.com/<username>/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS.git
cd IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS
```

### Prerequisites

- AWS Account (Free Tier eligible)
- AWS CLI installed and configured
- Git for version control
- Text editor (VS Code, Vim, etc.)

### Configure AWS CLI

```bash
# Configure AWS credentials
aws configure

# Verify configuration
aws sts get-caller-identity
```

## Lab 1: Account Security and IAM

### Create IAM Users with Policies

```bash
# Create a new IAM user
aws iam create-user --user-name security-admin

# Create a custom policy document
cat > security-admin-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:*",
        "cloudtrail:*",
        "cloudwatch:*"
      ],
      "Resource": "*"
    }
  ]
}
EOF

# Create and attach the policy
aws iam create-policy \
  --policy-name SecurityAdminPolicy \
  --policy-document file://security-admin-policy.json

aws iam attach-user-policy \
  --user-name security-admin \
  --policy-arn arn:aws:iam::${AWS_ACCOUNT_ID}:policy/SecurityAdminPolicy
```

### Enable MFA for Root Account

```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name root-mfa \
  --outfile mfa-qr.png \
  --bootstrap-method QRCodePNG

# Enable MFA (requires MFA codes from authenticator app)
aws iam enable-mfa-device \
  --user-name root \
  --serial-number arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/root-mfa \
  --authentication-code1 <CODE1> \
  --authentication-code2 <CODE2>
```

### Create IAM Groups with Least Privilege

```bash
# Create developer group
aws iam create-group --group-name Developers

# Attach managed policies
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

# Add user to group
aws iam add-user-to-group \
  --user-name security-admin \
  --group-name Developers
```

## Lab 2: Secure Isolation and Multitenancy

### Create Isolated VPC

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=SecureVPC}]'

# Create public subnet
aws ec2 create-subnet \
  --vpc-id <VPC_ID> \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id <VPC_ID> \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'

# Create internet gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=SecureIGW}]'

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id <IGW_ID> \
  --vpc-id <VPC_ID>
```

### Configure Security Groups

```bash
# Create web tier security group
aws ec2 create-security-group \
  --group-name web-tier-sg \
  --description "Security group for web tier" \
  --vpc-id <VPC_ID>

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id <SG_ID> \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Create database tier security group
aws ec2 create-security-group \
  --group-name db-tier-sg \
  --description "Security group for database tier" \
  --vpc-id <VPC_ID>

# Allow MySQL only from web tier
aws ec2 authorize-security-group-ingress \
  --group-id <DB_SG_ID> \
  --protocol tcp \
  --port 3306 \
  --source-group <WEB_SG_ID>
```

### Create Network ACLs

```bash
# Create network ACL
aws ec2 create-network-acl \
  --vpc-id <VPC_ID> \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=PrivateNACL}]'

# Add inbound rule
aws ec2 create-network-acl-entry \
  --network-acl-id <NACL_ID> \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 10.0.1.0/24 \
  --rule-action allow

# Add outbound rule
aws ec2 create-network-acl-entry \
  --network-acl-id <NACL_ID> \
  --egress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow
```

## Lab 3: Encryption and Key Management

### Create KMS Key

```bash
# Create customer managed key
aws kms create-key \
  --description "Data encryption key for Lab 3" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create alias
aws kms create-alias \
  --alias-name alias/lab3-encryption-key \
  --target-key-id <KEY_ID>

# Set key policy
cat > key-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::${AWS_ACCOUNT_ID}:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key for encryption",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::${AWS_ACCOUNT_ID}:user/security-admin"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws kms put-key-policy \
  --key-id <KEY_ID> \
  --policy-name default \
  --policy file://key-policy.json
```

### Encrypt S3 Bucket with KMS

```bash
# Create S3 bucket
aws s3api create-bucket \
  --bucket secure-data-bucket-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Enable default encryption with KMS
aws s3api put-bucket-encryption \
  --bucket secure-data-bucket-${AWS_ACCOUNT_ID} \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "alias/lab3-encryption-key"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Upload encrypted file
aws s3 cp sensitive-data.txt \
  s3://secure-data-bucket-${AWS_ACCOUNT_ID}/ \
  --server-side-encryption aws:kms \
  --ssekms-key-id alias/lab3-encryption-key
```

### Encrypt EBS Volume

```bash
# Create encrypted EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/lab3-encryption-key \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EncryptedVolume}]'

# Attach to EC2 instance
aws ec2 attach-volume \
  --volume-id <VOLUME_ID> \
  --instance-id <INSTANCE_ID> \
  --device /dev/sdf
```

### Encrypt Data at Application Level

```bash
# Encrypt plaintext using KMS
aws kms encrypt \
  --key-id alias/lab3-encryption-key \
  --plaintext fileb://plaintext.txt \
  --output text \
  --query CiphertextBlob > encrypted.bin

# Decrypt ciphertext
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode
```

## Lab 4: Access Control and Network Security

### Create VPC Endpoints

```bash
# Create S3 Gateway Endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id <VPC_ID> \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids <ROUTE_TABLE_ID>

# Create Interface Endpoint for Secrets Manager
aws ec2 create-vpc-endpoint \
  --vpc-id <VPC_ID> \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-east-1.secretsmanager \
  --subnet-ids <SUBNET_ID> \
  --security-group-ids <SG_ID>
```

### Configure AWS WAF

```bash
# Create IP set
aws wafv2 create-ip-set \
  --name BlockedIPs \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses 192.0.2.0/24 203.0.113.0/24

# Create web ACL
aws wafv2 create-web-acl \
  --name SecurityLabWAF \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://waf-rules.json \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=SecurityLabWAF
```

### Configure AWS Systems Manager Session Manager

```bash
# Create IAM role for EC2
cat > ec2-role-trust.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF

aws iam create-role \
  --role-name SSMRole \
  --assume-role-policy-document file://ec2-role-trust.json

aws iam attach-role-policy \
  --role-name SSMRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

# Start session without SSH
aws ssm start-session --target <INSTANCE_ID>
```

## Lab 5: Monitoring, Logging, and Incident Detection

### Enable CloudTrail

```bash
# Create S3 bucket for logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --region us-east-1

# Apply bucket policy
cat > trail-bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::cloudtrail-logs-${AWS_ACCOUNT_ID}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::cloudtrail-logs-${AWS_ACCOUNT_ID}/*",
      "Condition": {
        "StringEquals": {"s3:x-amz-acl": "bucket-owner-full-control"}
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --policy file://trail-bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name security-audit-trail \
  --s3-bucket-name cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --is-multi-region-trail \
  --enable-log-file-validation

aws cloudtrail start-logging --name security-audit-trail
```

### Configure CloudWatch Alarms

```bash
# Create SNS topic for alerts
aws sns create-topic --name SecurityAlerts

aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:${AWS_ACCOUNT_ID}:SecurityAlerts \
  --protocol email \
  --notification-endpoint ${ALERT_EMAIL}

# Create metric filter for unauthorized API calls
aws logs put-metric-filter \
  --log-group-name CloudTrail/DefaultLogGroup \
  --filter-name UnauthorizedAPICalls \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
    metricName=UnauthorizedAPICalls,metricNamespace=CloudTrailMetrics,metricValue=1

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICallsAlarm \
  --alarm-description "Triggers when unauthorized API calls are detected" \
  --metric-name UnauthorizedAPICalls \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:${AWS_ACCOUNT_ID}:SecurityAlerts
```

### Query CloudTrail Logs with Athena

```sql
-- Create Athena table for CloudTrail logs
CREATE EXTERNAL TABLE cloudtrail_logs (
  eventversion STRING,
  useridentity STRUCT<
    type:STRING,
    principalid:STRING,
    arn:STRING,
    accountid:STRING,
    username:STRING>,
  eventtime STRING,
  eventsource STRING,
  eventname STRING,
  awsregion STRING,
  sourceipaddress STRING,
  useragent STRING,
  errorcode STRING,
  errormessage STRING,
  requestparameters STRING,
  responseelements STRING
)
ROW FORMAT SERDE 'com.amazon.emr.hive.serde.CloudTrailSerde'
STORED AS INPUTFORMAT 'com.amazon.emr.cloudtrail.CloudTrailInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://cloudtrail-logs-${AWS_ACCOUNT_ID}/AWSLogs/${AWS_ACCOUNT_ID}/CloudTrail/';

-- Query failed login attempts
SELECT 
  eventtime,
  useridentity.username,
  sourceipaddress,
  errorcode,
  errormessage
FROM cloudtrail_logs
WHERE eventname = 'ConsoleLogin'
  AND errorcode IS NOT NULL
ORDER BY eventtime DESC
LIMIT 50;
```

### Configure AWS GuardDuty

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

# Get detector ID
DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)

# Create threat intelligence set
aws guardduty create-threat-intel-set \
  --detector-id ${DETECTOR_ID} \
  --name CustomThreatList \
  --format TXT \
  --location s3://threat-intel-bucket/threats.txt \
  --activate

# List findings
aws guardduty list-findings \
  --detector-id ${DETECTOR_ID} \
  --finding-criteria '{"Criterion":{"severity":{"Gte":7}}}'
```

## Common Patterns

### Secure EC2 Instance Launch

```bash
# Launch EC2 with security best practices
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --subnet-id <PRIVATE_SUBNET_ID> \
  --security-group-ids <RESTRICTED_SG_ID> \
  --iam-instance-profile Name=SSMRole \
  --metadata-options HttpTokens=required,HttpPutResponseHopLimit=1 \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/xvda",
      "Ebs": {
        "VolumeSize": 20,
        "VolumeType": "gp3",
        "Encrypted": true,
        "KmsKeyId": "alias/lab3-encryption-key",
        "DeleteOnTermination": true
      }
    }
  ]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=SecureInstance}]'
```

### Rotate IAM Access Keys

```bash
#!/bin/bash
# Rotate access keys for a user

USER_NAME="security-admin"

# Create new access key
NEW_KEY=$(aws iam create-access-key --user-name ${USER_NAME} --output json)
NEW_ACCESS_KEY=$(echo ${NEW_KEY} | jq -r '.AccessKey.AccessKeyId')
NEW_SECRET_KEY=$(echo ${NEW_KEY} | jq -r '.AccessKey.SecretAccessKey')

echo "New Access Key: ${NEW_ACCESS_KEY}"
echo "New Secret Key: ${NEW_SECRET_KEY}"

# Update AWS credentials file
aws configure set aws_access_key_id ${NEW_ACCESS_KEY} --profile ${USER_NAME}
aws configure set aws_secret_access_key ${NEW_SECRET_KEY} --profile ${USER_NAME}

# List old access keys
OLD_KEYS=$(aws iam list-access-keys --user-name ${USER_NAME} --query 'AccessKeyMetadata[?AccessKeyId!=`'${NEW_ACCESS_KEY}'`].AccessKeyId' --output text)

# Deactivate old keys (after testing new key)
for KEY in ${OLD_KEYS}; do
  aws iam update-access-key --user-name ${USER_NAME} --access-key-id ${KEY} --status Inactive
  echo "Deactivated old key: ${KEY}"
done
```

### Automated Security Group Auditing

```bash
#!/bin/bash
# Audit security groups for overly permissive rules

echo "Auditing Security Groups for 0.0.0.0/0 access..."

aws ec2 describe-security-groups --query 'SecurityGroups[*].[GroupId,GroupName,IpPermissions]' --output json | \
jq -r '.[] | select(.[2][]?.IpRanges[]?.CidrIp == "0.0.0.0/0") | "Security Group: \(.[1]) (\(.[0])) has unrestricted access"'
```

## Troubleshooting

### IAM Permission Errors

```bash
# Decode authorization failure message
aws sts decode-authorization-message \
  --encoded-message <ENCODED_MESSAGE> \
  --query DecodedMessage \
  --output text | jq '.'
```

### VPC Connectivity Issues

```bash
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=<VPC_ID>"

# Check NACLs
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=<VPC_ID>"

# Check security group rules
aws ec2 describe-security-groups --group-ids <SG_ID>

# Test connectivity with VPC Reachability Analyzer
aws ec2 create-network-insights-path \
  --source <SOURCE_ENI_ID> \
  --destination <DEST_ENI_ID> \
  --protocol tcp \
  --destination-port 443

aws ec2 start-network-insights-analysis \
  --network-insights-path-id <PATH_ID>
```

### KMS Key Access Issues

```bash
# List key grants
aws kms list-grants --key-id alias/lab3-encryption-key

# Get key policy
aws kms get-key-policy \
  --key-id alias/lab3-encryption-key \
  --policy-name default \
  --output text | jq '.'

# Check key state
aws kms describe-key --key-id alias/lab3-encryption-key
```

### CloudTrail Logging Not Working

```bash
# Get trail status
aws cloudtrail get-trail-status --name security-audit-trail

# Validate S3 bucket policy
aws s3api get-bucket-policy \
  --bucket cloudtrail-logs-${AWS_ACCOUNT_ID} \
  --output text | jq '.'

# Check CloudTrail event selectors
aws cloudtrail get-event-selectors --trail-name security-audit-trail
```

## Environment Variables

```bash
# Set AWS credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-east-1
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Lab-specific variables
export VPC_ID=vpc-xxxxx
export SUBNET_ID=subnet-xxxxx
export KMS_KEY_ALIAS=alias/lab3-encryption-key
export ALERT_EMAIL=security@example.com
```

## Documentation Best Practices

Each lab should include:

```markdown
# Lab X: [Title]

## Objective
Brief description of what you'll learn

## Prerequisites
- AWS account configured
- IAM permissions required
- Any other dependencies

## Implementation Steps

### Step 1: [Task Name]
\`\`\`bash
# Commands with explanations
\`\`\`

### Step 2: [Task Name]
Screenshots and evidence

## Verification
How to verify the implementation works

## Cleanup
Commands to remove resources and avoid charges

## Lessons Learned
Key takeaways and security insights

## References
- AWS documentation links
- Security best practices
```

## Best Practices

1. **Always use least privilege**: Grant minimum permissions required
2. **Enable MFA**: For all human users, especially privileged accounts
3. **Encrypt data**: At rest and in transit using KMS
4. **Use private subnets**: For resources that don't need internet access
5. **Enable logging**: CloudTrail, VPC Flow Logs, and application logs
6. **Automate security**: Use AWS Config rules and Security Hub
7. **Regular audits**: Review IAM policies, security groups, and access logs
8. **Tag resources**: For cost allocation and security tracking
9. **Use Systems Manager**: Instead of SSH for instance access
10. **Clean up resources**: Delete unused resources to avoid costs

## Additional Resources

- AWS Security Best Practices: https://aws.amazon.com/security/best-practices/
- AWS Well-Architected Security Pillar: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/
- AWS Security Hub: https://aws.amazon.com/security-hub/
- CIS AWS Foundations Benchmark: https://www.cisecurity.org/benchmark/amazon_web_services
