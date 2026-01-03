---
tags:
  - software-engineering
  - devops
  - serverless
  - cloud
created: 2026-01-02
status: 🔴
---
# ⚡ Serverless

> *"No server is easier to manage than no server."*

## 🎯 What is Serverless?

Serverless es un modelo de ejecución donde el cloud provider maneja dinámicamente la asignación de recursos. Tú escribes funciones, el provider se encarga del resto.

---

## 🏗️ Serverless Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   SERVERLESS ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    Events              Functions              Services          │
│  ┌─────────┐         ┌─────────┐           ┌─────────┐         │
│  │  HTTP   │────────►│         │──────────►│   DB    │         │
│  │ Request │         │         │           │         │         │
│  └─────────┘         │         │           └─────────┘         │
│  ┌─────────┐         │ Lambda  │           ┌─────────┐         │
│  │  Queue  │────────►│   /     │──────────►│  S3     │         │
│  │ Message │         │Function │           │ Storage │         │
│  └─────────┘         │         │           └─────────┘         │
│  ┌─────────┐         │         │           ┌─────────┐         │
│  │Schedule │────────►│         │──────────►│  API    │         │
│  │ (Cron)  │         │         │           │External │         │
│  └─────────┘         └─────────┘           └─────────┘         │
│                                                                 │
│  Trigger ─────────► Execute ─────────► Side Effects             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Serverless vs Containers vs VMs

| Aspect | Serverless | Containers | VMs |
|--------|------------|------------|-----|
| **Scaling** | Automatic (0 to ∞) | Manual/Auto | Manual |
| **Startup** | Cold: 100ms-1s | ~1s | Minutes |
| **Pricing** | Per invocation | Per container-hour | Per VM-hour |
| **Max runtime** | 15 min (Lambda) | Unlimited | Unlimited |
| **State** | Stateless | Stateful possible | Stateful |
| **Ops overhead** | Minimal | Medium | High |

---

## 💻 AWS Lambda Example

### Basic Handler
```typescript
// handler.ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';

export const handler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  try {
    const body = JSON.parse(event.body || '{}');
    
    // Process the request
    const result = await processRequest(body);
    
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify(result)
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' })
    };
  }
};

async function processRequest(data: any) {
  // Business logic here
  return { success: true, data };
}
```

### Event Sources
```typescript
// HTTP API (API Gateway)
export const httpHandler = async (event: APIGatewayProxyEvent) => {
  const { httpMethod, path, queryStringParameters, body } = event;
  // Handle HTTP request
};

// S3 Event
export const s3Handler = async (event: S3Event) => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = record.s3.object.key;
    // Process uploaded file
  }
};

// SQS Event
export const sqsHandler = async (event: SQSEvent) => {
  for (const record of event.Records) {
    const message = JSON.parse(record.body);
    // Process queue message
  }
};

// Scheduled Event (Cron)
export const cronHandler = async (event: ScheduledEvent) => {
  // Run scheduled task
};

// DynamoDB Stream
export const streamHandler = async (event: DynamoDBStreamEvent) => {
  for (const record of event.Records) {
    if (record.eventName === 'INSERT') {
      const newItem = record.dynamodb?.NewImage;
      // React to new item
    }
  }
};
```

---

## 🏭 Serverless Framework

### serverless.yml Configuration
```yaml
service: my-api

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
  stage: ${opt:stage, 'dev'}
  
  # Environment variables
  environment:
    TABLE_NAME: ${self:service}-${self:provider.stage}
    
  # IAM permissions
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:Query
            - dynamodb:Scan
            - dynamodb:GetItem
            - dynamodb:PutItem
            - dynamodb:UpdateItem
            - dynamodb:DeleteItem
          Resource:
            - !GetAtt UsersTable.Arn

functions:
  # HTTP API
  createUser:
    handler: src/handlers/users.create
    events:
      - http:
          path: users
          method: post
          cors: true
  
  getUser:
    handler: src/handlers/users.get
    events:
      - http:
          path: users/{id}
          method: get
          cors: true
  
  # Scheduled function
  dailyReport:
    handler: src/handlers/reports.daily
    events:
      - schedule: cron(0 9 * * ? *)  # 9 AM UTC daily
  
  # SQS consumer
  processOrders:
    handler: src/handlers/orders.process
    events:
      - sqs:
          arn: !GetAtt OrdersQueue.Arn
          batchSize: 10

resources:
  Resources:
    UsersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:provider.environment.TABLE_NAME}
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
    
    OrdersQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-orders-${self:provider.stage}

plugins:
  - serverless-esbuild
  - serverless-offline

custom:
  esbuild:
    bundle: true
    minify: false
```

---

## ❄️ Cold Starts

### Understanding Cold Starts
```
┌─────────────────────────────────────────────────────────────────┐
│                      COLD VS WARM START                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  COLD START (First invocation / After idle)                     │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ Download │ Start    │ Init     │ Handler  │            │    │
│  │ Code     │ Container│ Runtime  │ Execute  │   Total    │    │
│  │ ~100ms   │ ~200ms   │ ~100ms   │ ~50ms    │  ~450ms    │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  WARM START (Container reused)                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                              │ Handler  │            │      │
│  │        (cached)              │ Execute  │   Total    │      │
│  │                              │ ~50ms    │   ~50ms    │      │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cold Start Optimization
```typescript
// ❌ BAD - Initialize inside handler
export const handler = async (event) => {
  const db = new DatabaseClient(); // Cold start penalty
  const result = await db.query('SELECT * FROM users');
  return result;
};

// ✅ GOOD - Initialize outside handler
const db = new DatabaseClient(); // Reused across invocations

export const handler = async (event) => {
  const result = await db.query('SELECT * FROM users');
  return result;
};
```

### Provisioned Concurrency
```yaml
# Keep instances warm
functions:
  api:
    handler: handler.main
    provisionedConcurrency: 5  # Keep 5 instances warm
```

---

## 🎯 Serverless Patterns

### 1. API Backend
```
API Gateway ──► Lambda ──► DynamoDB
                  │
                  └──► S3 (files)
```

### 2. Event Processing
```
S3 Upload ──► Lambda ──► Process ──► SQS ──► Lambda ──► DB
```

### 3. Fan-out Pattern
```
                    ┌──► Lambda ──► Service A
SNS Topic ─────────┼──► Lambda ──► Service B
                    └──► Lambda ──► Service C
```

### 4. Saga Pattern (Distributed Transactions)
```
Step Functions:
┌────────┐    ┌────────┐    ┌────────┐
│ Create │───►│Reserve │───►│Charge  │
│ Order  │    │ Stock  │    │Payment │
└────────┘    └────────┘    └────────┘
     │             │             │
     ▼             ▼             ▼
  Compensate   Compensate   Compensate
  (on failure)
```

---

## ⚠️ Serverless Limitations

| Limitation | Impact | Mitigation |
|------------|--------|------------|
| **Cold starts** | Latency spike | Provisioned concurrency |
| **15 min timeout** | Long tasks fail | Use Step Functions |
| **Stateless** | No session | External storage |
| **Vendor lock-in** | Migration hard | Serverless Framework |
| **Debugging** | Complex locally | serverless-offline |
| **Cost at scale** | Can be expensive | Analyze breakeven |

---

## 💰 Cost Calculation

```
AWS Lambda Pricing (example):
- $0.20 per 1M requests
- $0.0000166667 per GB-second

Example calculation:
- 10M requests/month
- 512MB memory
- 200ms average duration

Compute cost:
= 10M × 0.2s × 0.5GB × $0.0000166667
= $16.67/month

Request cost:
= 10M × $0.20/1M
= $2.00/month

Total: ~$18.67/month

vs. EC2 t3.medium running 24/7:
= $30.37/month (on-demand)
```

---

## 📚 Related

- [[Programming/Software Engineering/DevOps/Cloud Fundamentals|Cloud Fundamentals]]
- [[Programming/Software Engineering/Architectural Patterns/Event-Driven Architecture|Event-Driven Architecture]]

---

← [[Programming/Software Engineering/DevOps/_Index|Back to DevOps]]
