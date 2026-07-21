# Introduction to Amazon API Gateway — Lab Notes

> **Lab:** SPL-58 · Version 2.0.36
> **Duration:** ~60 minutes
> **Prereq:** Completed *Introduction to AWS Lambda* self-paced lab

---

## 1. Lab Overview

Build a simple **FAQ micro-service** that returns a random question/answer pair via an **API Gateway** endpoint that invokes a **Lambda function**.

### Request Flow

```
User ──HTTP GET──▶ API Gateway ──JSON via VPC endpoint──▶ Lambda
User ◀──HTTP resp── API Gateway ◀──JSON via VPC endpoint── Lambda
```

- API Gateway transforms HTTP ⇄ JSON
- Lambda processes request and returns JSON
- Communication occurs through a VPC endpoint

### Objectives

- Create an AWS Lambda function
- Create an Amazon API Gateway endpoint
- Debug API Gateway + Lambda with **CloudWatch**

---

## 2. Services Used

### Amazon API Gateway
Managed service for creating, deploying, and maintaining APIs.

**Key features:**
- Transform request/response body & headers
- Access control via IAM
- API keys for third-party developers
- CloudWatch monitoring integration
- CloudFront response caching
- Multi-stage deployments (dev / test / prod)
- Custom domains
- Request/response models

### AWS Lambda
Serverless, event-driven compute — run code without managing servers.

**Key features:**
- Extend AWS services with custom logic
- Bring your own code / container images
- Automatic scaling & fault tolerance
- Connect to VPC, RDS, EFS
- IAM-integrated security
- Built-in monitoring & observability

---

## 3. Task 1 — Create the Lambda Function

### 3.1 Initial Setup

| Setting | Value |
|---|---|
| Function name | `FAQ` |
| Runtime | **Node.js 22.x** |
| Author | From scratch |
| Execution role | `lambda-basic-execution` (existing) |

**VPC configuration (required for this lab):**

| Field | Value |
|---|---|
| VPC | CIDR `10.0.0.0/16` |
| Subnets | `10.0.1.0/24` **and** `10.0.2.0/24` |
| Security group | contains `LambdaSecurityGroup` |

> ⏳ VPC-attached Lambda creation takes a few minutes. Wait for the success banner.

### 3.2 Function Code (`index.js`)

Replace the default handler:

```javascript
var json = {
  "service": "lambda",
  "reference": "https://aws.amazon.com/lambda/faqs/",
  "questions": [
    { "q": "What is AWS Lambda?", "a": "AWS Lambda lets you run code without provisioning or managing servers..." },
    { "q": "What events can trigger an AWS Lambda function?", "a": "DynamoDB updates, S3 object changes, CloudWatch logs, SES emails, SNS, Kinesis, Cognito sync, custom events, scheduled events..." },
    { "q": "When should I use AWS Lambda versus Amazon EC2?", "a": "EC2 = flexibility & control. Lambda = event-driven, zero provisioning, auto-scaling..." },
    { "q": "What kind of code can run on AWS Lambda?", "a": "Mobile backends, S3 object handlers, API call auditing, streaming data processing..." },
    { "q": "What languages does AWS Lambda support?", "a": "Node.js, Python, Java. Can launch processes in Bash, Go, Ruby via Amazon Linux." }
    // ... additional Q&A entries
  ]
};

export const handler = function (event, context) {
  var rand = Math.floor(Math.random() * json.questions.length);
  console.log("Quote selected: ", rand);

  var response = {
    body: JSON.stringify(json.questions[rand])
  };
  console.log(response);
  context.succeed(response);
};
```

**What it does:**
1. Defines a list of FAQs
2. Picks a random FAQ
3. Returns it as `{ body: "<json-string>" }`

Click **Deploy**.

### 3.3 Create API Gateway Trigger

**Configuration → General configuration → Edit:**
- **Description:** `Provide a random FAQ`

**Function overview → Add trigger:**

| Field | Value |
|---|---|
| Source | API Gateway |
| Intent | Create a new API |
| API type | **REST API** |
| Security | **Open** |
| API name | `FAQ-API` |
| Deployment stage | `myDeployment` |

---

## 4. Task 2 — Test the Function

### 4.1 Test via API Gateway URL
1. `Configuration → Triggers → API Gateway → Details`
2. Copy **API endpoint** → open in a new browser tab
3. Page displays a random FAQ:

```json
{
  "q": "What languages does AWS Lambda support?",
  "a": "AWS Lambda supports code written in Node.js, Python, and Java..."
}
```

### 4.2 Test via Lambda Console

| Field | Value |
|---|---|
| Event name | `BasicTest` |
| Payload | `{}` |

- **Save → Test**
- Expand **Execution result: succeeded → Details**
- Review **Summary** (duration, memory) and **Log output** (console logs / errors)

### 4.3 CloudWatch Logs
`Monitor tab → View CloudWatch logs → select a log stream` — shows the same event data plus `console.log` output.

---

## 5. RESTful API Design Reference

| Operation | URL | Function |
|---|---|---|
| GET | `/questions` | Return all questions |
| GET | `/questions/17` | Return question 17 |
| POST | `/questions` | Create a new question |
| PUT | `/questions/17` | Update question 17 |
| PATCH | `/questions/17` | Partially update question 17 |
| DELETE | `/questions/17` | Delete question 17 |

> Use **plural nouns** (`/questions`) with an identifier — never `/question/name`.

### REST Constraints (6)
1. Client-server separation
2. Stateless communication
3. Cacheable responses
4. Uniform interface
5. Layered system
6. Code-on-demand

---

## 6. API Gateway + Lambda Terminology

| Term | Meaning |
|---|---|
| **Resource** | URL endpoint/path (e.g. `/questions`) — a microservice |
| **Method** | Resource + HTTP verb (GET, POST, ...) |
| **Method Request** | Authorization, query params, headers from client |
| **Integration Request** | Backend target + mapping templates |
| **Integration Response** | Backend → API Gateway data mapping |
| **Method Response** | Response types, headers, content types to client |
| **Model** | JSON Schema defining request/response shape |
| **Stage** | Deployment path (dev / prod / v1) |
| **Blueprint** | Sample Lambda function used as a starting template |

---

## 7. Next Steps (Extend the Lab)

- Add **IAM authorization** to the endpoint
- Move FAQ data into **DynamoDB**
- Implement `/questions/{id}` for a specific question
- Implement `GET /questions` for all questions
- Implement `POST /questions` to add a question
- Try the **Serverless Framework**

---

## 8. Additional Resources
- [Amazon API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- Best Practices for a Pragmatic RESTful API
- Microservice architecture patterns
