# AWS Lambda — Lab Manual Study Notes

Step-by-step practical exercises covering Lambda basics, dynamic input, arithmetic operations, and S3 trigger integration.

---

## Lab 1: Hello World Lambda Function

**Objective:** Learn AWS Lambda basics.

**Steps:**
1. Log in to the AWS Console
2. Search **Lambda**
3. Click **Create Function**
4. Select **Author from scratch**
5. Function name: `HelloLambda`
6. Runtime: **Python 3.x**
7. Create a new role with basic Lambda permissions
8. Click **Create Function**

**Code:**
```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from AWS Lambda'
    }
```

---

## Lab 2: Dynamic Greeting Lambda

**Objective:** Take dynamic input from the user.

**Steps:**
1. Open the Lambda function
2. Replace the code (below)
3. Click **Deploy**
4. Create a **Test Event**
5. Run the test

**Code:**
```python
def lambda_handler(event, context):
    name = event['name']
    return {
        'statusCode': 200,
        'body': 'Hello ' + name
    }
```

**Test event example:**
```json
{
  "name": "Arpit"
}
```

---

## Lab 3: Calculator Lambda Function

**Objective:** Perform arithmetic operations using Lambda.

**Steps:**
1. Create a Lambda function
2. Paste the calculator code
3. Deploy the function
4. Create a **Test Event**
5. Execute the Lambda

**Code:**
```python
def lambda_handler(event, context):
    num1 = int(event['num1'])
    num2 = int(event['num2'])
    result = num1 + num2
    return {
        'result': result
    }
```

**Test event example:**
```json
{
  "num1": "10",
  "num2": "25"
}
```

> 💡 Extension idea: modify the code to accept an `operation` field (`add`, `subtract`, `multiply`, `divide`) and branch accordingly using `if`/`elif` statements.

---

## Lab 4: S3 File Upload Trigger

**Objective:** Trigger Lambda automatically when a file is uploaded to S3.

**Steps:**
1. Create an S3 bucket
2. Open the Lambda function
3. Click **Add Trigger**
4. Select **S3**
5. Select the bucket
6. Event type: **PUT**
7. Click **Add**
8. Upload a file to the bucket
9. Observe the Lambda execution (check **CloudWatch Logs**)

**Code:**
```python
def lambda_handler(event, context):
    print("File uploaded successfully")
    return {
        'statusCode': 200
    }
```

> 💡 To verify execution: open **CloudWatch → Log groups → /aws/lambda/[function-name]** and confirm the log entry appears after each upload.

---

## Interview Questions

1. **What is AWS Lambda?**
   A serverless compute service that runs code in response to events without requiring you to provision or manage servers — you pay only for the compute time consumed.

2. **What is serverless computing?**
   A cloud execution model where the cloud provider manages the infrastructure, automatically provisioning, scaling, and managing servers — developers focus purely on writing code.

3. **Difference between EC2 and Lambda?**

   | | EC2 | Lambda |
   |---|---|---|
   | Model | Provision and manage virtual servers | No servers to manage — fully serverless |
   | Billing | Pay for uptime (per second/hour running) | Pay only per invocation and execution duration |
   | Scaling | Manual or Auto Scaling groups | Automatic, event-driven scaling |
   | Execution | Long-running, persistent processes | Short-lived, stateless functions (max 15 min runtime) |
   | Use case | Full control over OS/runtime, long-running apps | Event-driven tasks, microservices, automation |

4. **What are Lambda triggers?**
   Event sources that invoke a Lambda function automatically — e.g., S3 (file upload), API Gateway (HTTP request), DynamoDB Streams, SNS, SQS, EventBridge (scheduled/rule-based events), CloudWatch Logs, etc.

5. **What is API Gateway?**
   A fully managed AWS service for creating, publishing, maintaining, monitoring, and securing APIs at any scale — commonly used as the HTTP-facing front door that invokes Lambda functions as backend logic.

---

### Quick Reference Summary
- **Lab progression:** static response → dynamic input → computation → event-driven trigger (S3)
- `event` = the input data passed into `lambda_handler`; `context` = runtime information about the invocation
- Always **Deploy** after editing code before testing
- Use **Test Events** to simulate different input payloads without needing a real trigger
- For S3-triggered functions, always verify via **CloudWatch Logs**, since there's no direct console output
- Lambda functions are inherently **stateless** — don't rely on data persisting between invocations