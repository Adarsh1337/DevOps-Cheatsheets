# AWS Cheatsheet

## Core Services Overview
- EC2: Virtual servers
- S3: Object storage
- IAM: Identity and permissions
- VPC: Virtual network
- RDS: Managed relational databases
- EKS: Managed Kubernetes
- Lambda: Serverless compute

## IAM
- Users, Groups, Roles, Policies (JSON)
- Least privilege, MFA, Access Keys rotation

### CLI Setup
```bash
aws configure
# Enter Access Key, Secret, region, output
```

### Common IAM CLI
```bash
aws iam list-users
aws iam create-user --user-name dev-user
aws iam attach-user-policy --user-name dev-user --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

## EC2

### Key commands
```bash
# List instance types
aws ec2 describe-instance-types --filters Name=free-tier-eligible,Values=true

# Launch instance
aws ec2 run-instances \
  --image-id ami-1234567890abcdef0 \
  --count 1 \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-1234567890abcdef0 \
  --subnet-id subnet-123456

# Describe instances
aws ec2 describe-instances --query 'Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,State:State.Name,AZ:Placement.AvailabilityZone}' --output table

# Stop/Start/Terminate
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

### Security Groups
- Stateful virtual firewalls per ENI
- Inbound/Outbound rules, use least privilege

## S3
```bash
# Create bucket
aws s3api create-bucket --bucket my-bucket --region us-east-1

# Upload/Sync
aws s3 cp file.txt s3://my-bucket/
aws s3 sync ./site s3://my-bucket/site

# Bucket policy example (public read)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

## VPC
- Components: VPC, Subnets (public/private), Route Tables, IGW, NAT GW, NACLs, SGs
- Typical 3-tier: ALB in public, ASG in private, RDS in private

### Create VPC (CLI)
```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-123 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-123 --internet-gateway-id igw-123
```

## RDS
```bash
# Create PostgreSQL instance
aws rds create-db-instance \
  --db-instance-identifier app-db \
  --engine postgres \
  --db-instance-class db.t3.micro \
  --allocated-storage 20 \
  --master-username admin \
  --master-user-password 'ChangeMe123!' \
  --publicly-accessible false

# List instances
aws rds describe-db-instances --query 'DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus}' --output table
```

## EKS
```bash
# Create cluster (eksctl)
eksctl create cluster --name app --region us-east-1 --nodes 2 --node-type t3.medium

# Update kubeconfig
aws eks update-kubeconfig --name app --region us-east-1

# Get nodes/pods
kubectl get nodes -o wide
kubectl get pods -A
```

## Lambda
```bash
# Create function (Python)
zip function.zip lambda_function.py
aws lambda create-function \
  --function-name hello \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-exec \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke
aws lambda invoke --function-name hello out.json
```

## Cost & Monitoring
- CloudWatch metrics/logs, Alarms, Dashboards
- CloudTrail for API auditing
- Budgets and Cost Explorer

## Best Practices
- Use IAM roles for EC2/Lambda
- Encrypt at rest (KMS) and in transit (TLS)
- Use SSM Parameter Store/Secrets Manager
- Tag resources consistently
- Use Infrastructure as Code (CloudFormation/Terraform)
