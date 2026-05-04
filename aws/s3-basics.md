# AWS S3 (Simple Storage Service)

**S3** is AWS's object storage service — infinitely scalable, highly durable, and one of the most versatile services in AWS. It's used for backups, static websites, data lakes, application assets, and much more.

---

## Core Concepts

### Buckets
- A **bucket** is a container for objects (files)
- Bucket names must be **globally unique** across all AWS accounts
- Buckets are created in a **specific region** (data stays there unless you replicate)
- There is no actual folder hierarchy — S3 is flat, but prefixes simulate folders

### Objects
- An object = a **file + metadata**
- Max object size: **5 TB**
- Max single PUT upload: **5 GB** (use Multipart Upload for anything over 100 MB)
- Each object has a **key** (its full "path"), e.g.: `photos/2026/vacation.jpg`

### S3 is Not a File System
- No true directories — `/` in key names is just a prefix convention
- No locking (unless you enable Object Lock)
- Strongly consistent for all operations (as of December 2020)

---

## Storage Classes

S3 offers multiple storage classes to balance cost vs. access frequency. You can set the class per object or automate transitions with Lifecycle Rules.

| Class | Use Case | Retrieval | Min Duration | Cost |
|---|---|---|---|---|
| **S3 Standard** | Frequently accessed data | Instant | None | Highest storage cost |
| **S3 Intelligent-Tiering** | Unknown or changing access patterns | Instant | None | Small monitoring fee |
| **S3 Standard-IA** | Infrequently accessed, rapid retrieval | Instant | 30 days | Lower storage, retrieval fee |
| **S3 One Zone-IA** | IA data that can be recreated | Instant | 30 days | ~20% cheaper than Standard-IA |
| **S3 Glacier Instant** | Archive with millisecond retrieval | Instant | 90 days | Very low storage |
| **S3 Glacier Flexible** | Archive, retrieval in minutes–hours | 1 min–12 hrs | 90 days | Lower storage |
| **S3 Glacier Deep Archive** | Long-term archive (7–10 years) | 12–48 hrs | 180 days | Cheapest storage |

> 💡 **One Zone-IA** stores data in a single AZ — it's cheaper but not resilient to AZ failure. Don't use it for data you can't recreate.

---

## Security

### Bucket Policies
- **Resource-based** JSON policies attached to the bucket
- Can grant access to other AWS accounts, IAM roles, or the public
- The primary way to make a bucket or objects **publicly accessible**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-public-bucket/*"
    }
  ]
}
```

### IAM Policies
- Identity-based policies attached to users/roles
- Control what IAM principals can do with S3
- Evaluated together with bucket policies (both must allow for cross-account)

### Block Public Access
- A **safety override** setting — blocks all public access regardless of bucket policy
- Enabled by default on all new buckets
- Can be set at the account level or per bucket
- Must be disabled before a bucket policy can grant public access

### ACLs (Access Control Lists)
- Legacy method of granting access — largely replaced by bucket policies
- AWS recommends disabling ACLs for most use cases (use bucket policies instead)
- Object ACLs can grant access to specific AWS accounts

### Access Points
- Named endpoints with their own access policies
- Simplify access management for large buckets with many users/apps
- Each access point can restrict access to a specific VPC

---

## Encryption

### Server-Side Encryption (SSE)
Data encrypted at rest by AWS. Three options:

| Type | Key managed by | Description |
|---|---|---|
| **SSE-S3** | AWS | AWS manages keys using AES-256; default on all new objects |
| **SSE-KMS** | AWS KMS | You control keys via KMS; audit trail via CloudTrail |
| **SSE-C** | You | You provide and manage the encryption key per request |

### Client-Side Encryption
- You encrypt data **before uploading** to S3
- AWS never sees the plaintext data

> 💡 **SSE-KMS** gives you an audit log of every key usage via CloudTrail — important for compliance.

### Encryption in Transit
- S3 supports HTTPS (TLS) for all requests
- You can enforce HTTPS-only access via a bucket policy using `aws:SecureTransport`

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"],
  "Condition": { "Bool": { "aws:SecureTransport": "false" } }
}
```

---

## Versioning

- Keeps **multiple versions** of an object in the same bucket
- Once enabled, versioning can only be **suspended**, not fully disabled
- Protects against accidental deletion — a delete just adds a **delete marker**
- To permanently delete a versioned object, you must delete the specific version ID
- Works with **MFA Delete** — requires MFA to permanently delete versions or change versioning state

---

## Replication

Replicates objects between buckets **asynchronously**. Versioning must be enabled on both source and destination.

| Type | Description |
|---|---|
| **CRR** (Cross-Region Replication) | Replicate to a bucket in a different region |
| **SRR** (Same-Region Replication) | Replicate within the same region |

Use cases: compliance, latency reduction, log aggregation, DR.

> ⚠️ Replication only applies to **new objects** after replication is enabled. Existing objects are not replicated automatically (use S3 Batch Operations for that).

---

## Lifecycle Rules

Automate transitioning objects between storage classes or expiring them.

```
Upload → S3 Standard (30 days)
       → S3 Standard-IA (60 days)
       → S3 Glacier Flexible (180 days)
       → Expire / Delete (365 days)
```

Rules can be applied to:
- All objects in a bucket
- A specific prefix (e.g., `logs/`)
- Objects with specific tags

---

## Performance

### Multipart Upload
- **Recommended** for objects > 100 MB; **required** for objects > 5 GB
- Uploads parts in parallel → faster uploads
- Resilient — failed parts can be retried individually

### S3 Transfer Acceleration
- Routes uploads through **CloudFront edge locations** to speed up long-distance transfers
- Uses the AWS backbone instead of the public internet
- Useful when users are geographically far from the bucket's region

### Byte-Range Fetches
- Download specific byte ranges of an object in parallel
- Speeds up large downloads and allows resuming interrupted transfers

### S3 Request Rate
- S3 supports **3,500 PUT/COPY/POST/DELETE** and **5,500 GET/HEAD** requests per second **per prefix**
- Spread objects across multiple prefixes to scale throughput
- Random prefixes (e.g., hash-based) used to avoid hot spots (less necessary today)

---

## S3 Select & Glacier Select
- Run **SQL queries directly on S3 objects** (CSV, JSON, Parquet)
- Retrieve only the data you need — reduces data transfer and cost
- Much faster than downloading the whole object and filtering client-side

---

## Static Website Hosting

S3 can host a static website:
- Enable static website hosting on the bucket
- Set an **index document** (e.g., `index.html`) and **error document**
- Make objects publicly accessible via a bucket policy
- Website URL format: `http://bucket-name.s3-website-region.amazonaws.com`

> 💡 For HTTPS on a static site, put **CloudFront** in front of the S3 bucket.

---

## Event Notifications

S3 can trigger actions when objects are created, deleted, or restored:

| Destination | Use Case |
|---|---|
| **SNS** | Fan-out notifications |
| **SQS** | Queue for downstream processing |
| **Lambda** | Serverless processing on upload |
| **EventBridge** | Advanced filtering and routing to many targets |

---

## Object Lock & Glacier Vault Lock

### S3 Object Lock
- Prevents objects from being **deleted or overwritten** for a fixed period or indefinitely
- Uses a **WORM** (Write Once, Read Many) model
- Two modes:
  - **Governance Mode** — users with special permissions can override
  - **Compliance Mode** — no one (including root) can delete until retention period expires
- Required for SEC 17a-4, FINRA, and similar compliance frameworks

### Legal Hold
- Prevents deletion indefinitely regardless of retention period
- Can be applied or removed by any user with `s3:PutObjectLegalHold` permission

---

## Pre-Signed URLs

- Generate a **temporary URL** that grants time-limited access to a private object
- Useful for sharing private files without making the bucket public
- Inherits the permissions of the IAM identity that generated it
- Expiry configurable (default: 1 hour, max varies)

```bash
aws s3 presign s3://my-bucket/my-file.pdf --expires-in 3600
```

---

## Key Exam/Interview Points

- S3 bucket names are **globally unique** but data is **region-specific**
- Max object size is **5 TB**; use Multipart Upload for > 5 GB
- **Block Public Access** overrides bucket policies — must be disabled for public access
- **Versioning cannot be fully disabled** once enabled — only suspended
- Replication requires versioning; only replicates **new** objects
- **SSE-KMS** provides CloudTrail audit logs for key usage
- **S3 Standard-IA and One Zone-IA** have a minimum 30-day storage charge
- **Glacier Deep Archive** is cheapest but retrieval takes 12–48 hours
- **Object Lock Compliance Mode** cannot be overridden by anyone, including root
- Pre-signed URLs give **temporary access** to private objects without changing bucket permissions
- S3 is **strongly consistent** for all operations (GET after PUT always returns the latest object)
- For HTTPS on static websites, use **CloudFront** — S3 website endpoints are HTTP only

---

*Last updated: May 2026*