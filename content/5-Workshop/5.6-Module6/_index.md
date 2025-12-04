---
title: "Module 6: Deployment & Operations"
date: "2025-01-15T14:00:00+07:00"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Overview

In this final module, you'll set up production deployment with CI/CD, implement monitoring and alerting, optimize costs, and apply security hardening to the Online Library platform. You'll also learn operational best practices for running serverless applications at scale.

**Duration:** ~90 minutes

**Services Used:**
- AWS Amplify (CI/CD)
- Amazon CloudWatch (monitoring, alarms)
- AWS CloudFormation (stack updates)
- AWS Budgets (cost alerts)
- AWS X-Ray (tracing)

---

## What You'll Learn

- Set up automated CI/CD with Amplify and CDK
- Configure CloudWatch dashboards and alarms
- Implement cost monitoring and optimization
- Apply security best practices and hardening
- Set up distributed tracing with X-Ray
- Create operational runbooks and documentation

---

## Architecture for This Module

```
GitHub → Amplify CI/CD → Build & Deploy Frontend
       ↓
       CDK Pipeline → Deploy Infrastructure
                   ↓
              CloudWatch ← Lambda Logs
                   ↓
              Alarms → SNS → Email/Slack
                   ↓
              X-Ray ← Trace Data
```

---

## Step 1: Set Up CI/CD with Amplify

### 1.1 Configure Amplify Build Settings

Update `amplify.yml`:

```yaml
# filepath: amplify.yml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
      - .next/cache/**/*
backend:
  phases:
    build:
      commands:
        - cd cdk
        - npm ci
        - npm run build
        - cdk deploy --require-approval never
```

### 1.2 Set Up Environment Variables

In Amplify Console, configure:

```bash
NEXT_PUBLIC_AWS_REGION=ap-southeast-1
NEXT_PUBLIC_USER_POOL_ID=<from-cdk-output>
NEXT_PUBLIC_USER_POOL_CLIENT_ID=<from-cdk-output>
NEXT_PUBLIC_API_URL=<from-cdk-output>
NEXT_PUBLIC_DISTRIBUTION_DOMAIN=<from-cdk-output>
```

### 1.3 Configure Branch Deployments

```typescript
// filepath: lib/amplify-stack.ts
import * as amplify from 'aws-cdk-lib/aws-amplify';

const app = new amplify.CfnApp(this, 'OnlineLibraryApp', {
  name: 'online-library',
  repository: 'https://github.com/your-org/online-library',
  accessToken: cdk.SecretValue.secretsManager('github-token'),
  buildSpec: amplify.BuildSpec.fromFile('amplify.yml'),
  environmentVariables: [
    {
      name: 'NEXT_PUBLIC_AWS_REGION',
      value: this.region,
    },
    // ...other variables
  ],
});

// Main branch for production
const mainBranch = new amplify.CfnBranch(this, 'MainBranch', {
  appId: app.attrAppId,
  branchName: 'main',
  enableAutoBuild: true,
  stage: 'PRODUCTION',
});

// Dev branch for staging
const devBranch = new amplify.CfnBranch(this, 'DevBranch', {
  appId: app.attrAppId,
  branchName: 'dev',
  enableAutoBuild: true,
  stage: 'DEVELOPMENT',
});
```

---

## Step 2: CloudWatch Monitoring

### 2.1 Create CloudWatch Dashboard

```typescript
// filepath: lib/monitoring-stack.ts
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';

export class MonitoringStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create dashboard
    const dashboard = new cloudwatch.Dashboard(this, 'OnlineLibraryDashboard', {
      dashboardName: 'online-library-metrics',
    });

    // Lambda metrics
    dashboard.addWidgets(
      new cloudwatch.GraphWidget({
        title: 'Lambda Invocations',
        left: [
          new cloudwatch.Metric({
            namespace: 'AWS/Lambda',
            metricName: 'Invocations',
            statistic: 'Sum',
            period: cdk.Duration.minutes(5),
          }),
        ],
      })
    );

    // API Gateway metrics
    dashboard.addWidgets(
      new cloudwatch.GraphWidget({
        title: 'API Requests',
        left: [
          new cloudwatch.Metric({
            namespace: 'AWS/ApiGateway',
            metricName: 'Count',
            statistic: 'Sum',
            period: cdk.Duration.minutes(5),
          }),
        ],
        right: [
          new cloudwatch.Metric({
            namespace: 'AWS/ApiGateway',
            metricName: '4XXError',
            statistic: 'Sum',
            period: cdk.Duration.minutes(5),
            color: cloudwatch.Color.ORANGE,
          }),
          new cloudwatch.Metric({
            namespace: 'AWS/ApiGateway',
            metricName: '5XXError',
            statistic: 'Sum',
            period: cdk.Duration.minutes(5),
            color: cloudwatch.Color.RED,
          }),
        ],
      })
    );

    // DynamoDB metrics
    dashboard.addWidgets(
      new cloudwatch.GraphWidget({
        title: 'DynamoDB Operations',
        left: [
          new cloudwatch.Metric({
            namespace: 'AWS/DynamoDB',
            metricName: 'ConsumedReadCapacityUnits',
            statistic: 'Sum',
            period: cdk.Duration.minutes(5),
          }),
          new cloudwatch.Metric({
            namespace: 'AWS/DynamoDB',
            metricName: 'ConsumedWriteCapacityUnits',
            statistic: 'Sum',
            period: cdk.Duration.minutes(5),
          }),
        ],
      })
    );
  }
}
```

### 2.2 Set Up CloudWatch Alarms

```typescript
// filepath: lib/alarms-stack.ts
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as subscriptions from 'aws-cdk-lib/aws-sns-subscriptions';
import * as actions from 'aws-cdk-lib/aws-cloudwatch-actions';

// Create SNS topic for alerts
const alertTopic = new sns.Topic(this, 'AlertTopic', {
  topicName: 'online-library-alerts',
  displayName: 'Online Library Alerts',
});

// Subscribe email
alertTopic.addSubscription(
  new subscriptions.EmailSubscription('ops@example.com')
);

// Lambda error rate alarm
const lambdaErrorAlarm = new cloudwatch.Alarm(this, 'LambdaErrorAlarm', {
  alarmName: 'online-library-lambda-errors',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/Lambda',
    metricName: 'Errors',
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 10,
  evaluationPeriods: 2,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});

lambdaErrorAlarm.addAlarmAction(new actions.SnsAction(alertTopic));

// API Gateway 5XX errors
const apiErrorAlarm = new cloudwatch.Alarm(this, 'ApiErrorAlarm', {
  alarmName: 'online-library-api-5xx-errors',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/ApiGateway',
    metricName: '5XXError',
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 5,
  evaluationPeriods: 1,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
});

apiErrorAlarm.addAlarmAction(new actions.SnsAction(alertTopic));

// DynamoDB throttling alarm
const dynamoThrottleAlarm = new cloudwatch.Alarm(this, 'DynamoThrottleAlarm', {
  alarmName: 'online-library-dynamodb-throttle',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/DynamoDB',
    metricName: 'UserErrors',
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 10,
  evaluationPeriods: 2,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
});

dynamoThrottleAlarm.addAlarmAction(new actions.SnsAction(alertTopic));
```

---

## Step 3: Cost Monitoring & Optimization

### 3.1 Set Up AWS Budgets

```typescript
// filepath: lib/budget-stack.ts
import * as budgets from 'aws-cdk-lib/aws-budgets';

const budget = new budgets.CfnBudget(this, 'MonthlyBudget', {
  budget: {
    budgetName: 'online-library-monthly-budget',
    budgetType: 'COST',
    timeUnit: 'MONTHLY',
    budgetLimit: {
      amount: 20, // $20/month
      unit: 'USD',
    },
  },
  notificationsWithSubscribers: [
    {
      notification: {
        notificationType: 'ACTUAL',
        comparisonOperator: 'GREATER_THAN',
        threshold: 80, // Alert at 80% of budget
      },
      subscribers: [
        {
          subscriptionType: 'EMAIL',
          address: 'billing@example.com',
        },
      ],
    },
    {
      notification: {
        notificationType: 'FORECASTED',
        comparisonOperator: 'GREATER_THAN',
        threshold: 100, // Alert if forecasted to exceed
      },
      subscribers: [
        {
          subscriptionType: 'EMAIL',
          address: 'billing@example.com',
        },
      ],
    },
  ],
});
```

### 3.2 Cost Optimization Strategies

**Lambda Optimization:**
- Set appropriate memory (128MB-512MB based on profiling)
- Use ARM64 (Graviton2) for 20% cost savings
- Implement caching to reduce invocations

**DynamoDB Optimization:**
- Use on-demand pricing for MVP
- Monitor and optimize GSI usage
- Implement caching with CloudFront

**S3 Optimization:**
- Lifecycle policies to delete old files
- Intelligent-Tiering for long-term storage
- Optimize object sizes

**CloudFront Optimization:**
- Cache API responses where appropriate
- Set appropriate TTL values
- Use compression

---

## Step 4: Security Hardening

### 4.1 IAM Least Privilege

Update Lambda execution roles:

```typescript
// filepath: lib/security-stack.ts
// Create specific role for each Lambda
const approveBookRole = new iam.Role(this, 'ApproveBookRole', {
  assumedBy: new iam.ServicePrincipal('lambda.amazonaws.com'),
  managedPolicies: [
    iam.ManagedPolicy.fromAwsManagedPolicyName(
      'service-role/AWSLambdaBasicExecutionRole'
    ),
  ],
});

// Grant only specific S3 permissions
uploadsBucket.grantRead(approveBookRole);
booksBucket.grantPut(approveBookRole);

// Grant only specific DynamoDB permissions
booksTable.grant(approveBookRole, 
  'dynamodb:GetItem',
  'dynamodb:UpdateItem'
);
```

### 4.2 Enable Encryption

```typescript
// filepath: lib/online-library-cdk-stack.ts
// S3 buckets with encryption
const uploadsBucket = new s3.Bucket(this, 'UploadsBucket', {
  encryption: s3.BucketEncryption.S3_MANAGED,
  enforceSSL: true, // Require HTTPS
});

// DynamoDB with encryption at rest
const booksTable = new dynamodb.Table(this, 'BooksTable', {
  encryption: dynamodb.TableEncryption.AWS_MANAGED,
});

// Enable CloudWatch Logs encryption
const logGroup = new logs.LogGroup(this, 'ApiLogs', {
  encryption: logs.Encryption.KMS_MANAGED,
});
```

### 4.3 API Security

```typescript
// filepath: lib/api-stack.ts
// Add throttling
const httpApi = new apigateway.HttpApi(this, 'OnlineLibraryApi', {
  throttle: {
    rateLimit: 100, // requests per second
    burstLimit: 200,
  },
});

// Add WAF (optional, for production)
const webAcl = new wafv2.CfnWebACL(this, 'ApiWaf', {
  scope: 'REGIONAL',
  defaultAction: { allow: {} },
  rules: [
    {
      name: 'RateLimitRule',
      priority: 1,
      statement: {
        rateBasedStatement: {
          limit: 2000,
          aggregateKeyType: 'IP',
        },
      },
      action: { block: {} },
      visibilityConfig: {
        sampledRequestsEnabled: true,
        cloudWatchMetricsEnabled: true,
        metricName: 'RateLimitRule',
      },
    },
  ],
  visibilityConfig: {
    sampledRequestsEnabled: true,
    cloudWatchMetricsEnabled: true,
    metricName: 'OnlineLibraryWaf',
  },
});
```

---

## Step 5: Distributed Tracing with X-Ray

### 5.1 Enable X-Ray for Lambda

```typescript
// filepath: lib/online-library-cdk-stack.ts
const createUploadUrlFunction = new lambda.Function(this, 'CreateUploadUrl', {
  runtime: lambda.Runtime.PYTHON_3_9,
  handler: 'index.lambda_handler',
  code: lambda.Code.fromAsset('lambda/createUploadUrl'),
  tracing: lambda.Tracing.ACTIVE, // Enable X-Ray
});
```

### 5.2 Add X-Ray SDK to Lambda

Update `lambda/createUploadUrl/requirements.txt`:

```
aws-xray-sdk
boto3
```

Update Lambda code:

```python
# filepath: lambda/createUploadUrl/index.py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch AWS SDK
patch_all()

@xray_recorder.capture('create_upload_url')
def lambda_handler(event, context):
    # Add custom metadata
    xray_recorder.put_metadata('user_id', user_id)
    xray_recorder.put_annotation('request_type', 'upload')
    
    # ...existing code...
```

---

## Step 6: Operational Runbooks

### 6.1 Incident Response Procedures

Create `docs/runbooks/incident-response.md`:

```markdown
# Incident Response Runbook

## High Error Rate

**Symptoms:** CloudWatch alarm for Lambda errors

**Investigation:**
1. Check CloudWatch Logs Insights:
```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

2. Check X-Ray service map for bottlenecks
3. Review recent deployments

**Resolution:**
- Rollback deployment if recent change
- Scale up Lambda memory if memory errors
- Check DynamoDB throttling

## Slow Response Times

**Symptoms:** Users reporting slow page loads

**Investigation:**
1. Check CloudFront cache hit ratio
2. Review Lambda duration metrics
3. Check DynamoDB performance metrics

**Resolution:**
- Increase Lambda memory
- Optimize DynamoDB queries
- Adjust CloudFront cache settings
```

### 6.2 Backup & Disaster Recovery

```typescript
// filepath: lib/backup-stack.ts
import * as backup from 'aws-cdk-lib/aws-backup';

// Create backup vault
const vault = new backup.BackupVault(this, 'BackupVault', {
  backupVaultName: 'online-library-backups',
});

// Create backup plan
const plan = new backup.BackupPlan(this, 'BackupPlan', {
  backupPlanName: 'daily-backups',
  backupPlanRules: [
    new backup.BackupPlanRule({
      ruleName: 'DailyBackup',
      scheduleExpression: cdk.aws_events.Schedule.cron({
        hour: '2',
        minute: '0',
      }),
      deleteAfter: cdk.Duration.days(30),
    }),
  ],
});

// Add DynamoDB table to backup
plan.addSelection('DynamoDBSelection', {
  resources: [
    backup.BackupResource.fromDynamoDbTable(booksTable),
  ],
});
```

---

## Verification & Testing

### Checklist

- CI/CD pipeline deploys automatically on push
- CloudWatch dashboard shows all metrics
- Alarms trigger and send notifications
- Budget alerts configured
- X-Ray traces appear in console
- IAM roles follow least privilege
- All data encrypted at rest and in transit
- Runbooks documented and tested

### Load Testing

```bash
# Install artillery
npm install -g artillery

# Create load test config
cat > load-test.yml <<EOF
config:
  target: "https://your-api-url"
  phases:
    - duration: 60
      arrivalRate: 10
  http:
    headers:
      Authorization: "Bearer YOUR_JWT_TOKEN"
scenarios:
  - name: "Search books"
    flow:
      - get:
          url: "/search?title=test"
EOF

# Run load test
artillery run load-test.yml
```

---

## Clean Up

### Complete Resource Cleanup

```bash
# Delete Amplify app
aws amplify delete-app --app-id YOUR_APP_ID

# Delete CDK stacks
cdk destroy --all

# Delete S3 buckets (if not auto-deleted)
aws s3 rb s3://your-bucket-name --force

# Delete CloudWatch log groups
aws logs delete-log-group --log-group-name /aws/lambda/function-name

# Verify all resources deleted
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=online-library
```

---

## Workshop Summary

Congratulations! You've completed the Online Library Workshop. You've built:

### Technical Achievements

**Authentication System** - Cognito with email verification and JWT  
**Upload Infrastructure** - Presigned URLs with progress tracking  
**Admin Approval Workflow** - Role-based access control  
**Content Delivery** - CloudFront with signed URLs and OAC  
**Search Functionality** - DynamoDB GSI with pagination  
**CI/CD Pipeline** - Automated deployments  
**Monitoring & Alerts** - CloudWatch dashboards and alarms  
**Security Hardening** - Encryption, IAM, and WAF  

### Skills Acquired

- Serverless architecture design and implementation
- AWS CDK infrastructure as code
- Frontend development with Next.js and React
- API design with API Gateway and Lambda
- Database design with DynamoDB
- Content delivery with S3 and CloudFront
- DevOps practices with CI/CD
- Operational excellence with monitoring and alerting

### Cost Summary

**Monthly operational cost (without Free Tier):** ~$9.80

The platform can scale to:
- **5,000 users:** ~$50/month
- **50,000 users:** ~$200/month

### Next Steps

**Enhance the Platform:**
1. Add file format validation (MIME type checking)
2. Implement automatic file deletion after 72 hours
3. Add bookmark and reading progress tracking
4. Build recommendation system
5. Add commenting and rating features
6. Implement analytics with QuickSight

**Learn More:**
- AWS Well-Architected Framework
- Advanced DynamoDB patterns
- CloudFront optimization techniques
- Security best practices for serverless

---

## Additional Resources

- [AWS Serverless Application Model](https://aws.amazon.com/serverless/sam/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Serverless Patterns Collection](https://serverlessland.com/patterns)
- [AWS CDK Workshop](https://cdkworkshop.com/)
- [AWS Security Best Practices](https://docs.aws.amazon.com/security/)

---

**Thank you for completing this workshop!** 🎉

You've successfully built a production-ready serverless application on AWS. Apply these skills to your own projects and continue exploring the AWS ecosystem.