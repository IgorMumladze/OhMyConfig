---
name: aws-deployment
description: Use when deploying applications to AWS cloud services or managing AWS infrastructure through CLI automation
---

# AWS Deployment Operations

## Overview

AWS CLI automation reduces deployment complexity and human error in cloud operations.

## When to Use

**Trigger symptoms:**
- Deployment failures or timeouts
- S3 upload/download issues
- Lambda function errors or version conflicts
- CloudWatch log access needs
- Infrastructure state inconsistencies
- Multi-environment deployment complexity

**Use cases:**
- Application deployment automation
- Infrastructure as code management
- Monitoring and log analysis
- Cross-environment synchronization
- Security and compliance automation

## Core Pattern

### Deployment Flow
```bash
# Verify → Deploy → Validate → Monitor
aws s3 ls                    # Verify bucket access
aws s3 sync ./dist s3://bucket/  # Deploy
aws cloudfront create-invalidation  # Clear cache
aws logs tail /aws/lambda/function  # Monitor
```

### Infrastructure Management Flow
```bash
# Plan → Apply → Verify → Monitor
aws cloudformation validate template.yaml
aws cloudformation deploy --template-file template.yaml --stack-name prod
aws cloudformation describe-stacks --stack-name prod
```

## Quick Reference

| Service | Operation | Command | Verification |
|---------|-----------|---------|-------------|
| S3 Upload | `aws s3 sync ./build s3://bucket/` | `aws s3 ls s3://bucket/` |
| Lambda Deploy | `aws lambda update-function-code` | `aws lambda invoke --function-name` |
| CloudFront Cache | `aws cloudfront create-invalidation` | `aws cloudfront get-distribution` |
| EC2 Management | `aws ec2 describe-instances` | `aws ec2 start-instances` |
| CloudWatch Logs | `aws logs tail /aws/lambda/function` | Error pattern detection |

## Implementation

### S3 Deployment Strategy
```bash
# Atomic deployments with rollback capability
CURRENT_VERSION=$(aws s3api get-bucket-versioning --bucket my-app | jq -r '.Status')
if [ "$CURRENT_VERSION" = "Enabled" ]; then
    aws s3 sync ./dist s3://my-app/ --delete
    aws s3api put-bucket-versioning --bucket my-app --versioning-configuration Status=Enabled
fi
```

### Lambda Blue-Green Deployment
```bash
# Create alias, shift traffic gradually
NEW_VERSION=$(aws lambda publish-version --function-name my-function)
aws lambda create-alias --function-name my-function --name prod --function-version $NEW_VERSION
# Gradually shift traffic: 10% → 50% → 100%
for percent in 10 50 100; do
    aws lambda update-alias --function-name my-function --name prod --routing-config "additionalVersionWeights={$NEW_VERSION=$percent}"
    sleep 60  # Monitor between shifts
done
```

### Infrastructure Monitoring
```bash
# Set up automated alerts
aws cloudwatch put-metric-alarm \
    --alarm-name "LambdaErrorRate" \
    --metric-name Errors \
    --namespace AWS/Lambda \
    --statistic Sum \
    --period 300 \
    --threshold 10 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2
```

## Common Mistakes

| Issue | Cause | Fix |
|--------|--------|-----|
| Access denied | Incorrect IAM permissions | Use least-privilege principle, verify roles |
| S3 sync timeouts | Large files, poor connection | Use multipart upload, check region |
| Lambda cold starts | No provisioned concurrency | Set appropriate concurrency limits |
| CloudFormation drift | Manual changes outside IaC | Use drift detection, automated remediation |
| Cost overruns | Unmonitored resources | Set budgets, regular reviews |

## Real-World Impact

- **Deployment time**: 5 minutes vs 45 minutes manual
- **Rollback capability**: 30 seconds vs 30 minutes manual
- **Error detection**: Immediate vs hours of log digging  
- **Infrastructure consistency**: 100% vs variable manual configuration
- **Team productivity**: Consistent deployments vs knowledge silos