---
title: "Blog 3"
date: "2025-09-09T15:44:00+07:00"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Getting Started with Healthcare Data Lakes: Using Microservices

Data lakes can help hospitals and healthcare facilities transform data into business insights and maintain business continuity while protecting patient privacy. A **data lake** is a centralized, managed, and secure repository for storing all your data, both in raw and processed form for analysis. Data lakes allow you to break down data silos and combine different types of analytics to gain insights and make better business decisions.

This blog post is part of a larger series on getting started with healthcare data lakes. In my last blog post in the series, *"Getting Started with Healthcare Data Lakes: Deep Dive into Amazon Cognito"*, I focused on specific details of using Amazon Cognito and Attribute Based Access Control (ABAC) to authenticate and authorize users in the healthcare data lake solution. In this blog, I detail how the solution has evolved at a fundamental level, including the design decisions I made and additional features used. You can access the code samples for the solution at this Git repo for reference.

---

## Architecture Guidance

The main change since the last presentation of the overall architecture is separating the single service into a set of smaller services to improve maintainability and flexibility. Integrating a large volume of different healthcare data often requires specialized connectors for each format; by keeping them separately packaged as microservices, we can add, remove, and modify each connector without affecting others. The microservices are loosely coupled through centralized publish/subscribe messaging in what I call the "pub/sub hub".

This solution represents what I would consider another reasonable sprint iteration from my last post. The scope remains limited to simple ingestion and parsing of **HL7v2 messages** formatted according to **Encoding Rule 7 (ER7)** through a REST interface.

**The solution architecture is now as follows:**

> *Figure 1. Overall architecture; colored boxes represent separate services.*

---

Although the term *microservices* has some inherent ambiguity, several characteristics are common:
- They are small, autonomous, loosely coupled
- Reusable, communicate through well-defined interfaces
- Specialized to solve one thing
- Often implemented in **event-driven architecture**

When determining where to create boundaries between microservices, consider:
- **Intrinsic**: technology used, performance, reliability, scalability
- **External**: dependent functionality, frequency of change, reusability
- **Human**: team ownership, managing *cognitive load*

---

## Technology Selection and Communication Scope

| Communication Scope | Technologies / Patterns to Consider |
| --- | --- |
| Within a microservice | Amazon Simple Queue Service (Amazon SQS), AWS Step Functions |
| Between microservices in a service | AWS CloudFormation cross-stack references, Amazon Simple Notification Service (Amazon SNS) |
| Between services | Amazon EventBridge, AWS Cloud Map, Amazon API Gateway |

---

## The Pub/Sub Hub

Using a **hub-and-spoke** architecture (or message broker) works well with a small number of closely related microservices.
- Each microservice only depends on the *hub*
- Connections between microservices are limited only to message content
- Reduces synchronous calls since pub/sub is asynchronous one-way *push*

Disadvantage: requires **coordination and monitoring** to prevent microservices from processing wrong messages.

---

## Core Microservice

Provides foundational data and communication layer, including:
- **Amazon S3** bucket for data
- **Amazon DynamoDB** for data catalog
- **AWS Lambda** to write messages to data lake and catalog
- **Amazon SNS** topic as *hub*
- **Amazon S3** bucket for artifacts like Lambda code

> Only allows indirect write access to data lake via Lambda function → ensures consistency.

---

## Front Door Microservice

- Provides API Gateway for external REST interaction
- Authentication & authorization based on **OIDC** through **Amazon Cognito**
- Self-managed *deduplication* mechanism using DynamoDB instead of SNS FIFO because:
  1. SNS deduplication TTL is only 5 minutes
  2. SNS FIFO requires SQS FIFO
  3. Proactively informs sender that message is duplicate

---

## Staging ER7 Microservice

- Lambda "trigger" subscribes to pub/sub hub, filters messages by attribute
- Step Functions Express Workflow to convert ER7 → JSON
- Two Lambdas:
  1. Fix ER7 format (newline, carriage return)
  2. Parsing logic
- Results or errors are pushed back to pub/sub hub

---

## New Features in Solution

### 1. AWS CloudFormation Cross-Stack References

Example *outputs* in core microservice:

```yaml
Outputs:
  Bucket:
    Value: !Ref Bucket
    Export:
      Name: !Sub ${AWS::StackName}-Bucket
  ArtifactBucket:
    Value: !Ref ArtifactBucket
    Export:
      Name: !Sub ${AWS::StackName}-ArtifactBucket
  Topic:
    Value: !Ref Topic
    Export:
      Name: !Sub ${AWS::StackName}-Topic
  Catalog:
    Value: !Ref Catalog
    Export:
      Name: !Sub ${AWS::StackName}-Catalog
  CatalogArn:
    Value: !GetAtt Catalog.Arn
    Export:
      Name: !Sub ${AWS::StackName}-CatalogArn
```

Other stacks can reference these exports using `!ImportValue`. This creates dependencies between stacks - you cannot delete the exporting stack while other stacks are importing its values.

### 2. Amazon SNS Message Filtering

The pub/sub hub uses SNS topic subscriptions with **filter policies** to route messages to appropriate microservices. Each Lambda "trigger" subscribes with specific attribute filters.

Example filter policy:

```json
{
  "messageType": ["ER7"],
  "action": ["parse", "validate"]
}
```

This ensures each microservice only receives relevant messages, reducing unnecessary invocations and improving efficiency.

### 3. AWS Step Functions Express Workflows

Used in the staging ER7 microservice for orchestrating the conversion pipeline. Express Workflows are optimized for:
- High-volume, short-duration workflows
- Event-driven processing
- Lower cost compared to Standard Workflows

The workflow coordinates:
1. Format validation and correction
2. ER7 to JSON parsing
3. Error handling and retries
4. Publishing results back to hub

### 4. Amazon DynamoDB Deduplication

Custom deduplication implementation using DynamoDB with:
- Message hash as partition key
- TTL (Time To Live) attribute for automatic cleanup
- Conditional writes to detect duplicates atomically

Advantages over SNS FIFO deduplication:
- Configurable retention period (not limited to 5 minutes)
- Can provide feedback to sender
- Independent of queue type

### 5. Lambda Trigger Pattern

Each microservice uses Lambda functions that:
- Subscribe to SNS topic with filter policies
- Process messages asynchronously
- Publish results/errors back to hub
- Maintain loose coupling between services

This event-driven pattern allows:
- Independent scaling of each microservice
- Easy addition of new microservices
- Fault isolation
- Simplified debugging and monitoring

---

## Architecture Evolution Insights

**Benefits Realized:**
- **Flexibility**: Can modify or replace individual microservices without affecting others
- **Scalability**: Each microservice scales independently based on its workload
- **Maintainability**: Smaller, focused codebases easier to understand and modify
- **Testability**: Each service can be tested in isolation
- **Deployability**: Independent deployment of services reduces risk

**Challenges Encountered:**
- **Complexity**: More moving parts require better monitoring and observability
- **Coordination**: Need clear contracts between services
- **Testing**: Integration testing becomes more complex
- **Debugging**: Distributed tracing essential for troubleshooting

---

## Best Practices Applied

1. **Single Responsibility**: Each microservice handles one specific domain
2. **Loose Coupling**: Services communicate only through pub/sub hub
3. **Event-Driven**: Asynchronous processing improves resilience
4. **Infrastructure as Code**: All resources defined in CloudFormation
5. **Monitoring**: CloudWatch logs and metrics for each service
6. **Security**: Least privilege IAM roles, encryption at rest and in transit

---

## Future Enhancements

- Add more healthcare data format connectors (FHIR, DICOM)
- Implement dead letter queues for failed messages
- Add API versioning and rate limiting
- Implement distributed tracing with AWS X-Ray
- Add automated testing pipeline
- Implement canary deployments for safer rollouts

---

## Conclusion

The evolution from monolithic to microservices architecture has significantly improved the healthcare data lake solution's maintainability and flexibility. The pub/sub hub pattern provides effective service coordination while maintaining loose coupling. AWS services like SNS, Lambda, and Step Functions make implementing event-driven microservices straightforward and cost-effective.

The modular design allows teams to work independently on different data connectors, accelerating development and reducing risk. As healthcare data integration requirements grow, this architecture can easily accommodate new formats and processing logic without disrupting existing functionality.
