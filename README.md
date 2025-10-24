# AWS Resource Auto Tagging Automation

> Automate the tagging of newly created AWS resources using **CloudTrail**, **EventBridge**, and **Lambda**. This solution ensures that all supported AWS services are automatically tagged at creation time for compliance, cost allocation, and governance consistency.

---

## 1. Overview

- **Template Type**: AWS CloudFormation (YAML)
- **Core Components**:
  - **CloudTrail (multi-region)**: Captures create events
  - **EventBridge Rules**: Filters relevant events (e.g., `Create*`, `RunInstances`) and triggers Lambda
  - **Lambda (Python 3.8)**: Parses events and applies tags using **Resource Groups Tagging API**
  - **IAM Role & Policy**: Grants least privilege for tagging actions across multiple services
  - **S3 Bucket**: Stores CloudTrail logs with lifecycle cleanup rules

- **Use Cases**: Migration Acceleration Program (MAP/MAP2.0), cost attribution, compliance enforcement, multi-account governance

---

## 2. Architecture Workflow

1. A new AWS resource (EC2, RDS, S3, EKS, etc.) is created.
2. **CloudTrail** logs the creation event.
3. **EventBridge** detects the event and triggers the **Lambda** function.
4. **Lambda** extracts the resource ARN from the event payload.
5. **Lambda** calls **Resource Groups Tagging API** to apply the predefined tags.

This event-driven architecture provides near real-time tagging across all supported regions.

---

## 3. Supported Services and Events

| Service | Event Example |
|----------|----------------|
| EC2 / EBS / VPC | `RunInstances`, `CreateVolume`, `CreateVpc`, `CreateNatGateway`, etc. |
| ELB | `CreateLoadBalancer` |
| RDS | `CreateDBInstance` |
| DMS | `CreateReplicationInstance` |
| ElastiCache | `CreateReplicationGroup` |
| EKS | `CreateCluster` |
| S3 | `CreateBucket` |
| Lambda | `CreateFunction20150331` |
| DynamoDB | `CreateTable` |
| KMS | `CreateKey` |
| SNS | `CreateTopic` |
| SQS | `CreateQueue` |
| EFS | `CreateMountTarget` |
| OpenSearch | `CreateDomain` |

---

## 4. Template Parameters

| Parameter | Description | Example |
|------------|-------------|----------|
| `AutomationTags` | JSON string of tags to apply | `{"owner":"platform","env":"prod"}` |
| `LambdaAutoTaggingFunctionName` | Lambda function name | `aws-tagging-automation-function` |
| `EventBridgeRuleName` | Rule name | `aws-tagging-automation-rules` |
| `IAMAutoTaggingRoleName` | IAM Role name | `aws-tagging-automation-role` |
| `IAMAutoTaggingPolicyName` | IAM Policy name | `aws-tagging-automation-policy` |
| `TrailName` | CloudTrail name | `aws-tagging-automation-trail` |
| `BucketName` | CloudTrail S3 bucket prefix | `aws-tagging-automation-bucket` |

Final S3 bucket name format: `<BucketName>-<Region>-<AccountId>`.

---

## 5. Deployment Steps

### 5.1 AWS Console
1. Go to **CloudFormation** → **Create Stack (with new resources)**.
2. Upload `(Global)-auto-tagging-automation-v1.1.yaml`.
3. Fill in required parameters including `AutomationTags`.
4. Acknowledge IAM creation capability.
5. Deploy the stack.

### 5.2 AWS CLI
```bash
aws cloudformation deploy \
  --stack-name tagging-automation \
  --template-file (Global)-auto-tagging-automation-v1.1.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    AutomationTags='{"owner":"platform","env":"prod"}' \
    LambdaAutoTaggingFunctionName=aws-tagging-automation-function \
    EventBridgeRuleName=aws-tagging-automation-rules \
    IAMAutoTaggingRoleName=aws-tagging-automation-role \
    IAMAutoTaggingPolicyName=aws-tagging-automation-policy \
    TrailName=aws-tagging-automation-trail \
    BucketName=aws-tagging-automation-bucket
```

---

## 6. Verification

- **Option A:** Create a simple resource (e.g., EC2 instance, S3 bucket, or SNS topic)
- **Option B:** Check **CloudWatch Logs** for Lambda execution output
- **Option C:** Confirm tags appear in the AWS console under resource Tags tab

The Lambda automatically tags associated EBS volumes created with EC2 instances.

---

## 7. Security & Permissions

- IAM Role grants minimal permissions for tagging (`tag:TagResources`, `tag:GetResources`, etc.) and service-specific `Describe*` calls.
- S3 bucket and CloudTrail policies restrict access to the CloudTrail service principal with `bucket-owner-full-control`.

**Recommendations:**
- Refine IAM policy scope for production environments.
- Enable KMS encryption for Lambda environment variables.
- Add CloudWatch alarms for Lambda failures.
- Combine with AWS **Tag Policies** and **Service Control Policies (SCP)** for full compliance.

---

## 8. Cost Considerations

- **CloudTrail (multi-region)**: Management events are free; data events incur cost.
- **S3**: Minimal storage cost with 30-day lifecycle cleanup.
- **Lambda / EventBridge / CloudWatch Logs**: Negligible operational cost.

Total cost impact is minimal compared to the value of consistent cost allocation and compliance.

---

## 9. Known Limitations

- Some AWS services may have event structure variations (e.g., OpenSearch ARN capitalization).
- Long-running creation events (e.g., DynamoDB tables) may delay tagging.
- This solution only applies to **new** resources. Use AWS Tag Editor for retroactive tagging.

---

## 10. FAQ

**Q1: Can I extend it to more services?**  
Yes. Add event patterns in EventBridge and corresponding handler logic in the Lambda code.

**Q2: Can the tag values be dynamic?**  
Yes. Modify the Lambda to extract contextual metadata from the event payload or fetch from AWS Systems Manager Parameter Store or DynamoDB.

**Q3: How do I use this in multiple accounts?**  
Deploy the same stack across accounts or use AWS **StackSets** for organization-wide rollout.

---

## 11. Cleanup

Deleting the CloudFormation stack removes all deployed resources, including EventBridge, Lambda, IAM roles, and CloudTrail.  
If the S3 bucket still contains logs, empty it before deleting.

---

## 12. Version

- Template: `(Global)-auto-tagging-automation-v1.1.yaml`

---

## 13. About Dependix

**Dependix** is an **AWS Partner** based in **Kuala Lumpur, Malaysia**, specializing in **cloud migration (MAP)**, **cost optimization**, **platform engineering**, and **compliance & governance**.  

Our mission is to empower organizations to manage their AWS infrastructure with automation, visibility, and cost-efficiency. Dependix helps you navigate the cloud seas with confidence. ⛵
