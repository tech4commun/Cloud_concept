# Introduction to Amazon S3 — Lab Guide

**Duration:** ~1 hour
**Scenario:** An EC2 app needs to push daily report data to S3. Only that EC2 instance should be able to write to the bucket, and files need protection against accidental deletion.

## Objectives
- Create a bucket in Amazon S3
- Add an object to a bucket
- Manage access permissions on an object and a bucket
- Create a bucket policy
- Use bucket versioning

---

## Task 1: Create a Bucket

1. Go to **S3** in the AWS Console → **Create bucket**.
2. Copy your **AWSLabsUser Account ID** (top-right dropdown → Account ID).
3. **Bucket name:** `reportbucket-<Account ID>` (must be globally unique, 3–63 chars, lowercase/numbers/hyphens only, can't be changed later).
4. **Object Ownership:** select **ACLs enabled** → **Object writer**.
5. Leave **Region** at default.
6. Scroll down → **Create bucket**.

✅ **Done:** Bucket created.

---

## Task 2: Upload an Object to the Bucket

1. Download `new-report.png` locally.
2. Open your `reportbucket-...` in the S3 Console.
3. **Upload → Add files** → select `new-report.png` → **Upload**.
4. Confirm the green **Upload succeeded** banner → **Close**.

✅ **Done:** Object uploaded.

---

## Task 3: Make an Object Public

### Confirm it's private by default
1. Open `new-report.png` → copy its **Object URL**.
2. Paste into a new browser tab → **Access Denied** (expected — S3 objects are private by default).

### Try making it public via ACL (blocked)
3. On the object page → **Object actions → Make public using ACL** → fails with a warning: *Public access is blocked* (because **Block Public Access (BPA)** is enabled at the bucket level).
   > Bucket-level BPA settings override individual object permissions.

### Turn off Block Public Access at the bucket level
4. Go to bucket → **Permissions** tab → **Block public access (bucket settings) → Edit**.
5. Deselect **Block all public access** (leave the individual sub-options deselected for now).
6. **Save changes** → type `confirm` → **Confirm**.

### Now make the object public
7. Go back to **Objects** tab → `new-report.png` → **Object actions → Make public using ACL** → **Make public**.
8. Refresh the tab with the earlier Access Denied error → image now loads.

> This grants access to **one object only**. To grant access to a whole bucket, use a **bucket policy** (next task).

✅ **Done:** Object made publicly readable.

---

## Task 4: Test Connectivity from the EC2 Instance

1. Go to **EC2 → Instances (running)** → select **Bastion Host** → **Connect**.
2. **Connection method:** Session Manager → **Connect**.
   > Session Manager uses HTTPS (443) — no need to open SSH port 22.
3. In the session:
   ```bash
   cd ~
   pwd          # should show /home/ssm-user
   aws s3 ls    # lists your S3 buckets, including reportbucket-...
   aws s3 ls s3://reportbucket<NUMBER>   # lists new-report.png
   cd reports
   ls           # shows local test files
   aws s3 cp report-test1.txt s3://reportbucket<NUMBER>
   ```
4. This **upload fails** — the EC2 instance's role currently has **read-only** S3 access (no `PutObject` permission).

> Note: An **Instance Profile** (identity) + **Role** (permissions) were pre-attached to this EC2 instance, currently allowing only list/read actions.

✅ **Done:** Confirmed EC2 can read but not write to S3 yet.

---

## Task 5: Create a Bucket Policy

Goal: let the EC2 role both **read and write** to the bucket.

1. Download `sample-file.txt` locally → upload it to your bucket (same process as Task 2).
2. Copy its **Object URL** → open in a new tab → **Access Denied** (expected).
3. Go to **IAM → Roles** → search `EC2InstanceProfileRole` → copy its **Role ARN**
   (e.g. `arn:aws:iam::123456789012:role/EC2InstanceProfileRole`).
4. Go back to your S3 bucket → **Permissions** tab → **Bucket Policy → Edit**.
5. Copy the **Bucket ARN** shown below the policy editor (e.g. `arn:aws:s3:::reportbucket987987`).

### Build the policy with AWS Policy Generator
6. Open the [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html) in a new tab.
7. Configure:
   - **Type of Policy:** S3 Bucket Policy
   - **Effect:** Allow
   - **Principal:** paste the `EC2InstanceProfileRole` ARN
   - **Actions:** `PutObject`, `GetObject`
   - **ARN:** paste the Bucket ARN + append `/*` → e.g. `arn:aws:s3:::reportbucket987987/*`
8. **Add Statement** → **Generate Policy** → copy the resulting JSON:
   ```json
   {
       "Version": "2012-10-17",
       "Id": "Policy1604361694227",
       "Statement": [
           {
               "Sid": "Stmt1604361692117",
               "Effect": "Allow",
               "Principal": {
                   "AWS": "arn:aws:iam::123456789012:role/EC2InstanceProfileRole"
               },
               "Action": ["s3:GetObject", "s3:PutObject"],
               "Resource": "arn:aws:s3:::reportbucket987987/*"
           }
       ]
   }
   ```
9. Paste into the **Bucket policy editor** → **Save changes**.

### Retest from EC2
10. Back in the SSM session:
    ```bash
    pwd    # /home/ssm-user/reports
    aws s3 ls s3://reportbucket<NUMBER>
    ls
    aws s3 cp report-test1.txt s3://reportbucket<NUMBER>     # now succeeds
    aws s3 ls s3://reportbucket<NUMBER>                      # confirms upload
    aws s3 cp s3://reportbucket<NUMBER>/sample-file.txt sample-file.txt   # download works too
    ls
    ```

✅ **Done:** EC2 can now `PutObject` and `GetObject` via the bucket policy.

### Bonus: allow public read too
- Refreshing the `sample-file.txt` Access Denied tab still fails — the policy only grants access to the EC2 role.
- **Try it yourself:** add a second statement granting `GetObject` to Principal `*` (everyone). Example:
  ```json
  {
      "Version": "2012-10-17",
      "Id": "Policy1604428844058",
      "Statement": [
          {
              "Sid": "Stmt1604428821481",
              "Effect": "Allow",
              "Principal": {"AWS": "arn:aws:iam::123456789012:role/EC2InstanceProfileRole"},
              "Action": ["s3:GetObject", "s3:PutObject"],
              "Resource": "arn:aws:s3:::reportbucket987987/*"
          },
          {
              "Sid": "Stmt1604428842806",
              "Effect": "Allow",
              "Principal": "*",
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::reportbucket987987/*"
          }
      ]
  }
  ```
- Refresh the browser tab — text should now load.

---

## Task 6: Explore Versioning

Goal: protect files against accidental deletion.

1. Bucket → **Properties** tab → **Bucket Versioning → Edit** → **Enable** → **Save changes**.
   > Versioning applies to the **entire bucket**, not individual objects. Has cost implications (stores every version).
2. Download the (updated) `sample-file.txt` — same filename, new content — and upload it to the bucket.
3. Refresh the tab showing the old `sample-file.txt` content — new text appears.
   > S3 always serves the **latest version** unless a specific version is requested.

### View old versions
4. Bucket → **Objects** tab → toggle **Show versions** on.
5. Open `sample-file.txt` → **Versions** tab → select the bottom version (`null`) → **Open** → see original content.
   > Note: `new-report.png` shows only one version (`null`) since it was uploaded *before* versioning was enabled.
   > Directly opening an old version's **Object URL** gives Access Denied — the bucket policy only allows the *latest* version. Accessing older versions via URL requires adding `s3:GetObjectVersion` to the policy.

### Test delete protection
6. Turn **Show versions** off → select `sample-file.txt` → **Delete** → type `delete` → confirm.
   - Object disappears from the default view.
7. Turn **Show versions** back on → object reappears with a **Delete marker** as the newest "version"; older versions remain intact.
   > With versioning on, S3 never truly deletes on a normal delete — it just adds a delete marker as the new "current" version.
8. Select the **Delete marker** version → **Delete** → type `permanently delete` → confirm.
   - This removes the delete marker → object is **restored** to its prior visible state.

### Delete a specific version permanently
9. With **Show versions** on, select a specific version → **Delete** → type `permanently delete` → confirm.
   - This time the object is truly gone for that version — **no new delete marker is created** when deleting a specific version.
10. Confirm via the Object URL that the remaining version's content loads correctly.

✅ **Done:** Versioning enabled and tested — including recovering from an accidental delete.

---

## Summary

| Step | Outcome |
|---|---|
| 1 | Created a uniquely-named S3 bucket |
| 2 | Uploaded an object |
| 3 | Learned objects are private by default; BPA overrides object-level ACLs |
| 4 | Verified EC2 role initially had read-only S3 access |
| 5 | Used a bucket policy to grant the EC2 role read/write access |
| 6 | Enabled versioning; tested delete markers vs. permanent version deletion |

## Additional Resources
- [Amazon S3](https://aws.amazon.com/s3/)
- [Amazon S3 bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)
- [Amazon S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Amazon Resource Names (ARNs) and AWS Service Namespaces](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html)
- [AWS JSON Policy Elements](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html)
- [Actions, Resources, and Condition Keys for Amazon S3](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazons3.html)
- [Amazon S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [Undelete objects in Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RestoringPreviousVersions.html)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)