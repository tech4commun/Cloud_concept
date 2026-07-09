
# AWS S3 Static Website Hosting Guide
### Production Deployment Blueprint via AWS Console & CLI

**Executive Summary:** Hosting static frontend content directly out of Amazon S3 cuts server operating costs to near zero while leveraging a globally resilient backbone. This guide covers setting up an S3 bucket, writing a bucket policy for public reads, enabling static website hosting, and deploying files via both the AWS Console and CLI.

> **Architecture Note**
> S3 static hosting serves text files, CSS, images, asset packages, and pure JavaScript (React, Vue, Vanilla JS). It does **not** run dynamic server-side runtimes — no Node.js backend controllers, Python/PHP handlers, or embedded relational databases.

---

## Section 1: Core Front-End File (`index.html`)

A standalone demo dashboard page (`CloudCore Analytics`) styled with flexbox/CSS grid, used to verify deployment. Key structural elements:

- `<style>` block using CSS custom properties (`--primary`, `--accent`, `--text`, `--bg`, `--card-bg`)
- Header with logo + "Live on S3" status badge
- Hero section with heading and description
- A card grid showing mock metrics: Storage Infrastructure, Availability Map, Delivery Latency, Monthly Cost
- A mock animated bar chart (`.chart-mock`, `.bar`)
- Footer with copyright

> Full HTML/CSS source is provided in the original PDF (Pages 2–3) — save it locally as `index.html` before uploading.

---

## Section 2: Implementation via AWS Management Console (GUI)

### Step 1 — Provision the Bucket
1. Log into the AWS Management Console.
2. Search for **S3** in the services dashboard.
3. Select **Create bucket**.
4. Enter a globally unique, lowercase name (e.g., `web-dashboard-2026-demo`).
5. Under **Block Public Access settings**, uncheck the master "block all" box.
6. Acknowledge the warning about relaxed access rules, then click **Create bucket**.

### Step 2 — Enable Static Website Hosting
1. Open the bucket → **Properties** tab.
2. Scroll to **Static website hosting** → click **Edit**.
3. Set the toggle to **Enable**.
4. In **Index document**, enter: `index.html`
5. Leave the error document blank (optional) → **Save changes**.

### Step 3 — Define Public Read Access
1. Go to the **Permissions** tab.
2. Under **Bucket policy**, click **Edit**.
3. Paste the following policy (update the bucket name in the ARN):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::web-dashboard-2026-demo/*"
    }
  ]
}
```

4. Click **Save changes**.

### Step 4 — Upload the Object
1. Go to the **Objects** tab → click **Upload**.
2. Add your local `index.html` file → run the upload.
3. Return to **Properties** and click the generated **Bucket website endpoint** link to view the live site.

---

## Section 3: Automation via AWS CLI

Run these commands sequentially from a configured AWS CLI shell.

**Step 1 — Create the bucket**
```bash
aws s3api create-bucket --bucket web-dashboard-2026-demo --region us-east-1
```

**Step 2 — Disable Block Public Access**
```bash
aws s3api put-public-access-block --bucket web-dashboard-2026-demo \
  --public-access-block-configuration \
  "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```

**Step 3 — Apply the public-read bucket policy**

Save the JSON from Section 2 as `policy.json`, then run:
```bash
aws s3api put-bucket-policy --bucket web-dashboard-2026-demo --policy file://policy.json
```

**Step 4 — Enable website hosting**
```bash
aws s3api put-bucket-website --bucket web-dashboard-2026-demo \
  --website-configuration '{"IndexDocument": {"Suffix": "index.html"}}'
```

**Step 5 — Upload files & get the endpoint**
```bash
# Sync/upload the file
aws s3 cp index.html s3://web-dashboard-2026-demo/

# Resulting website endpoint:
# http://web-dashboard-2026-demo.s3-website-us-east-1.amazonaws.com
```

---

## Section 4: Troubleshooting / Verification Checklist

| Issue | Likely Cause | Fix |
|---|---|---|
| **HTTP 403 Forbidden** | Block Public Access still enabled, or bucket name in policy ARN doesn't match exactly | Verify BPA settings; double-check `arn:aws:s3:::bucket-name/*` matches the real bucket name |
| **HTTP 404 Not Found** | Uploaded file name doesn't match the index document setting (e.g., `Index.html` vs `index.html`) | Rename file / re-upload with correct casing |
| **CLI `AccessDenied`** | IAM identity lacks `s3:PutBucketPolicy` (or related) permissions | Grant the required IAM permissions to the executing user/role |

---

## Cleanup — Decommissioning the Environment

> ⚠️ **Warning:** Leaving public test buckets running creates security drift and possible cost overhead. Remove them when done testing.

```bash
aws s3 rm s3://web-dashboard-2026-demo --recursive
aws s3 rb s3://web-dashboard-2026-demo
```

---

### Quick Reference Summary
- **Two settings required for a public static site:** (1) disable Block Public Access, (2) add a public-read bucket policy — enabling website hosting alone does *not* make objects public.
- **Index document** must match the uploaded filename exactly (case-sensitive).
- **Endpoint format:** `http://<bucket-name>.s3-website-<region>.amazonaws.com`
- S3 static hosting = **frontend only** — no server-side code execution.