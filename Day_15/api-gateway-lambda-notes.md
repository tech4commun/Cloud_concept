# API Gateway + Lambda — Complete Step-by-Step Guide (AWS Free Tier)

> Build a REST API using **Amazon API Gateway** that invokes an **AWS Lambda** function and returns a JSON response.

---

## 🎯 Objective

Create a public REST endpoint that triggers a Lambda function and returns structured JSON data — using only AWS Free Tier services.

---

## 🏗️ Architecture

```
Browser / Postman  →  API Gateway (HTTP API)  →  AWS Lambda  →  JSON Response
```

| Component     | Role                                          |
| ------------- | --------------------------------------------- |
| API Gateway   | Public HTTPS entry point, routes requests     |
| Lambda        | Serverless compute, executes business logic   |
| JSON Response | Returned to client with HTTP `statusCode 200` |

---

## 🪜 Step-by-Step Setup

### Step 1 — Login
- Sign in to the **AWS Management Console**.
- Select the **same region** for all services (e.g. `eu-north-1`).

### Step 2 — Create Lambda Function
1. Search **Lambda** → **Create Function**.
2. Choose **Author from scratch**.
3. Function Name: `HelloAPI`
4. Runtime: **Python 3.x**
5. Leave the **default execution role**.
6. Click **Create Function**.

### Step 3 — Add Code

Replace the default handler with:

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": "Hello from Lambda via API Gateway!"})
    }
```

- Click **Deploy**.
- Click **Test** → create a sample event → verify **execution succeeded**.

### Step 4 — Create API Gateway
1. Search **API Gateway** → **Create API**.
2. Choose **HTTP API** → **Build**.

### Step 5 — Add Integration
- Integration type: **Lambda**
- Select function: `HelloAPI`
- Click **Next**.

### Step 6 — Configure Route
| Setting | Value    |
| ------- | -------- |
| Method  | `GET`    |
| Path    | `/hello` |

Click **Next**.

### Step 7 — Configure Stage
- Stage Name: `prod`
- Click **Create / Deploy**.

### Step 8 — Copy Invoke URL
Copy the generated API endpoint, e.g.:
```
https://abc123xyz.execute-api.eu-north-1.amazonaws.com
```

### Step 9 — Test the API
Open in browser or Postman:
```
<InvokeURL>/hello
```
Expected response:
```json
{ "message": "Hello from Lambda via API Gateway!" }
```

### Step 10 — Modify Response
- Update the `message` in Lambda code.
- Click **Deploy** → refresh the browser to see the new response.

---

## ⚠️ Common Errors

| Error                       | Cause / Fix                                        |
| --------------------------- | -------------------------------------------------- |
| **500 Internal Server Error** | Check Lambda **CloudWatch logs**                 |
| **403 Forbidden**           | Verify **deployment** and **permissions**          |
| **404 Not Found**           | Verify the **route path** (`/hello`)               |
| **Wrong Region**            | Ensure Lambda + API Gateway are in the **same region** |

---

## 🧪 Student Exercises

1. Return **your name**.
2. Return your **college name**.
3. Return the **current date/time**.
4. Return a JSON object containing **name, course, and city**.
5. Create an additional route `/welcome`.

<details>
<summary>💡 Hint: current date/time handler</summary>

```python
import json
from datetime import datetime

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps({"now": datetime.utcnow().isoformat() + "Z"})
    }
```
</details>

---

## 🚀 Next Lab

**API Gateway → Lambda → DynamoDB CRUD API**
Extend this setup to persist and retrieve data from a DynamoDB table.
