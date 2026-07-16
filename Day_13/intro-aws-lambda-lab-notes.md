# Introduction to AWS Lambda — Lab Notes

**Lab:** SPL-88 v2.3.34 · **Duration:** 45 min
**Goal:** Build a serverless image thumbnail app using S3 + Lambda + CloudWatch.

---

## 1. Overview

AWS Lambda = event-driven compute service. Runs code in response to events (S3 upload, DynamoDB change, API call, IoT signal, etc.) with automatic resource management. No servers to provision.

### Objectives
- Create a Lambda function
- Configure an S3 bucket as a Lambda event source
- Trigger Lambda by uploading an object to S3
- Monitor Lambda via CloudWatch Logs

---

## 2. Architecture / Flow

```
User ──upload──▶ S3 (source bucket)
                     │  ObjectCreated event
                     ▼
                  AWS Lambda (Create-Thumbnail)
                     │  reads object, resizes via Pillow
                     ▼
               S3 (target bucket: <source>-resized)
                     │
                     ▼
              CloudWatch Logs (metrics + logs)
```

**Event flow:**
1. User uploads object → source S3 bucket
2. S3 detects `ObjectCreated` event
3. S3 invokes Lambda, passing event data (bucket + key)
4. Lambda executes handler
5. Handler downloads image → resizes → uploads to `-resized` bucket

---

## 3. Task 1 — Create S3 Buckets

Create **two** buckets (names must be globally unique):

| Purpose | Example name |
|---|---|
| Source (uploads) | `images-123456789` |
| Target (thumbnails) | `images-123456789-resized` |

- Keep all other settings default.
- Download `HappyFace.jpg` (1280×853) and upload to the **source** bucket.

> Note: bucket names are region-independent but must be globally unique.

---

## 4. Task 2 — Create Lambda Function

**Basic info**
- Name: `Create-Thumbnail`
- Runtime: **Python 3.12**
- Execution role: **existing** `lambda-execution-role` (grants S3 read/write)

**Networking (lab-specific requirement)**
- VPC: CIDR `10.0.0.0/16`
- Subnet: CIDR `10.0.1.0/24`
- Security group: the one named `...LambdaSecurityGroup...`

> Note: Normally Lambda does NOT need a VPC. Required here for lab isolation. VPC-attached Lambdas take a few minutes to create.

**Add trigger**
- Source: **S3**
- Bucket: source `images-...` bucket
- Event type: **All object create events**
- Acknowledge recursive invocation warning ✅

**Upload code**
- Code tab → Upload from → Amazon S3 location
- Paste `AmazonS3LinkURL` (from lab panel) → Save

**Runtime settings**
- Handler: `CreateThumbnail.handler` ← must match `<filename>.<function>`

**General configuration**
- Description: `Create a thumbnail-sized image`
- Memory → more memory = more CPU
- Timeout → max execution duration

---

## 5. Lambda Function Code (reference)

```python
import boto3
import os
import sys
import uuid
from PIL import Image
import PIL.Image

s3_client = boto3.client('s3')

def resize_image(image_path, resized_path):
    with Image.open(image_path) as image:
        image.thumbnail((128, 128))
        image.save(resized_path)

def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key    = record['s3']['object']['key']
        download_path = '/tmp/{}{}'.format(uuid.uuid4(), key)
        upload_path   = '/tmp/resized-{}'.format(key)

        s3_client.download_file(bucket, key, download_path)
        resize_image(download_path, upload_path)
        s3_client.upload_file(upload_path, '{}-resized'.format(bucket), key)
```

**What it does:**
1. Reads `Records` from the S3 event (bucket + key).
2. Downloads object to `/tmp/` (Lambda's ephemeral 512 MB scratch).
3. Resizes to 128×128 thumbnail using **Pillow**.
4. Uploads result to `<bucket>-resized`.

---

## 6. Task 3 — Test the Function

Create test event:
- Name: `Upload`
- Template: **S3 Put**
- In JSON, replace `example-bucket` (appears **twice** — name + ARN) with your source bucket.
- Replace `test%2Fkey` with `HappyFace.jpg`.

Click **Test** → expect `Executing function: succeeded`.

**Verify:** open `<bucket>-resized` in S3 → download `HappyFace.jpg` → confirm it is a small thumbnail.

---

## 7. Task 4 — Monitoring & Logging (CloudWatch)

**Monitor tab metrics:**
| Metric | Meaning |
|---|---|
| Invocations | Times function ran |
| Duration | Avg / min / max execution time |
| Error count & success rate | Failures vs successes |
| Throttles | Hit concurrency limit (default 1000) |
| Async delivery failures | Errors writing to destination / DLQ |
| Iterator Age | Age of last record (Kinesis / DynamoDB streams) |
| Concurrent executions | Active function instances |

**Logs:** Monitor → **View CloudWatch logs** → open log stream → expand entries.

Each log entry includes: `RequestId`, `Duration`, `Billed Duration` (rounded up to 100 ms), `Memory Size`, `Max Memory Used`, plus any `print()` output.

---

## 8. Setup Checklist

- [ ] Create source bucket `images-<rand>`
- [ ] Create target bucket `images-<rand>-resized`
- [ ] Upload `HappyFace.jpg` to source
- [ ] Create Lambda `Create-Thumbnail` (Python 3.12)
- [ ] Attach existing role `lambda-execution-role`
- [ ] Configure VPC / subnet / security group (lab)
- [ ] Add S3 trigger (all object create events)
- [ ] Upload `CreateThumbnail.zip` from S3 URL
- [ ] Set handler = `CreateThumbnail.handler`
- [ ] Create S3 Put test event → Test
- [ ] Confirm thumbnail in `-resized` bucket
- [ ] Inspect CloudWatch Logs stream

---

## 9. Key Takeaways / Tips

- **Handler format** = `<python_filename_without_.py>.<function_name>`. Wrong value → `handler not found`.
- **`/tmp/`** is the only writable path (512 MB default, up to 10 GB configurable).
- **Memory ↑ = CPU ↑** — tune for image processing speed.
- **Billed duration** is rounded up to nearest 1 ms (100 ms in older accounts).
- **Recursive invocation risk:** never write output to the same bucket that triggers the function → infinite loop.
- **Pillow** must be packaged with the deployment zip (or use a Lambda layer) — it is not in the default runtime.
- **VPC-attached Lambda** = slower cold starts + needs NAT gateway for internet access.
- Extendable ideas: multiple thumbnail sizes, WebP conversion, watermarking, SNS notification on completion.
