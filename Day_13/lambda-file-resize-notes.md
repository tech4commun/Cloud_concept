# AWS Lambda Project — Resize Uploaded File to 50KB or 50MB

> Automatically resize/compress files uploaded to Amazon S3 using AWS Lambda.

---

## 📌 Overview

- **Trigger:** File upload (PUT) to a source S3 bucket
- **Action:** AWS Lambda downloads, resizes/compresses, and re-uploads
- **Destination:** A separate S3 bucket for optimized files
- **Runtime:** Python (with Pillow)

---

## 🧱 Architecture Flow

```text
User Upload
    │
    ▼
[ Source S3 Bucket ] ──(PUT event)──► [ AWS Lambda ] ──► [ Destination S3 Bucket ]
                                            │
                                            ▼
                                     [ CloudWatch Logs ]
```

---

## 🪜 Step-by-Step Setup

### Step 1 — Create S3 Buckets
Create **two** buckets in Amazon S3:
- `source-bucket` → original uploaded files
- `destination-bucket` → resized / compressed files

### Step 2 — Create IAM Role
Create an IAM Role for Lambda with permissions for:
- `AmazonS3FullAccess` (or scoped read/write on the two buckets)
- `CloudWatchLogsFullAccess`

### Step 3 — Create Lambda Function
- Go to **AWS Lambda → Create Function**
- Runtime: **Python**
- Attach the IAM Role from Step 2

### Step 4 — Install Required Libraries
- Use **Pillow (PIL)** for image compression / resizing
- Add dependencies via **Lambda Layer** or a **ZIP deployment package**

### Step 5 — Add Lambda Trigger
- Source: `source-bucket`
- Event type: **ObjectCreated (PUT)**

### Step 6 — Write Lambda Code
Function logic:
1. Download uploaded file from source bucket
2. Resize / compress it
3. Upload optimized version to destination bucket

### Step 7 — Upload Test File
Upload any image into the **source** bucket to trigger Lambda.

### Step 8 — Verify Output
Check the **destination** bucket for the resized/compressed file.

---

## 🐍 Sample Lambda Code (Python — Image Compression)

```python
import boto3
from PIL import Image
import os

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    download_path = '/tmp/' + key
    upload_path = '/tmp/resized-' + key

    # Download original from source bucket
    s3.download_file(bucket, key, download_path)

    # Compress / resize
    image = Image.open(download_path)
    image.save(upload_path, optimize=True, quality=40)

    # Upload to destination bucket
    s3.upload_file(upload_path, 'destination-bucket-name', 'resized-' + key)

    return {
        'statusCode': 200,
        'body': 'File resized successfully'
    }
```

> **Note:** Change the `quality` value (and/or add `image.thumbnail((w, h))`) to hit the target size such as **50KB** or **50MB**.

---

## 🎯 Tuning Tips: Hitting 50KB vs 50MB

| Target Size | Strategy |
|-------------|----------|
| ~50 KB      | Low `quality` (e.g. 30–50), downscale dimensions with `image.thumbnail()` |
| ~50 MB      | High `quality` (85–95), keep original dimensions, or convert to lossless PNG/TIFF |

Iterate: save → check `os.path.getsize(upload_path)` → adjust quality in a loop until within target range.

---

## ✅ Checklist

- [ ] Source & destination S3 buckets created
- [ ] IAM role with S3 + CloudWatch permissions
- [ ] Lambda function created with Python runtime
- [ ] Pillow added via Layer or ZIP
- [ ] S3 PUT trigger wired to Lambda
- [ ] Test upload succeeds
- [ ] Resized file appears in destination bucket
- [ ] CloudWatch logs show successful execution
