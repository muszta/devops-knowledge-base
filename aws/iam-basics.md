# AWS IAM Basics

AWS **Identity and Access Management (IAM)** is a global service that lets you securely control who can access your AWS resources and what they can do with them.

---

## Core Concepts

### Users
An IAM **User** represents a person or application that interacts with AWS. Each user has:
- A unique name within your AWS account
- Long-term credentials (password and/or access keys)
- No permissions by default — permissions must be explicitly granted

### Groups
A **Group** is a collection of IAM users. Groups make it easy to manage permissions for multiple users at once. Key points:
- A user can belong to multiple groups
- Groups cannot be nested (no groups within groups)
- Permissions assigned to a group apply to all its members

### Roles
An IAM **Role** is an identity with specific permissions that can be *assumed* by trusted entities (users, services, or external accounts). Unlike users, roles have no permanent credentials — they issue temporary security tokens. Common use cases:
- Granting an EC2 instance access to S3
- Cross-account access
- Federated identity (SSO, OIDC, SAML)

### Policies
A **Policy** is a JSON document that defines permissions. It specifies what **actions** are allowed or denied on which **resources** and under what **conditions**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

#### Policy Types

| Type | Description |
|---|---|
| **AWS Managed** | Pre-built by AWS, updated automatically |
| **Customer Managed** | Created and managed by you |
| **Inline** | Embedded directly in a single user, group, or role |
| **Resource-based** | Attached to a resource (e.g., S3 bucket policy) |
| **Permission Boundaries** | Limits the maximum permissions an identity can have |
| **SCPs (Service Control Policies)** | Applied at the AWS Organizations level |

---

## How Permissions Are Evaluated

IAM follows a strict evaluation logic:

1. **Explicit Deny** — always wins; overrides everything
2. **Explicit Allow** — grants access if no deny is present
3. **Implicit Deny** — the default; everything is denied unless explicitly allowed

> 💡 **Remember:** In IAM, if it's not explicitly allowed, it's denied.

---

## Key Principles

### Least Privilege
Grant only the permissions required to perform a task — nothing more. Start with minimal permissions and add more as needed.

### Root Account
The root account has unlimited access to everything in your AWS account. Best practices:
- Do **not** use it for day-to-day tasks
- Enable MFA on the root account immediately
- Create an admin IAM user for administrative tasks

### MFA (Multi-Factor Authentication)
Add an extra layer of security by requiring a second factor (virtual MFA app, hardware key, or SMS) when signing in or performing sensitive actions.

---

## IAM Best Practices

- ✅ Enable MFA for all users, especially root
- ✅ Follow the principle of **least privilege**
- ✅ Use **roles** for applications and AWS services (never embed access keys in code)
- ✅ Rotate access keys regularly
- ✅ Use **groups** to assign permissions, not individual users
- ✅ Use **IAM Access Analyzer** to identify unintended external access
- ✅ Monitor activity with **AWS CloudTrail**
- ❌ Never share access keys or credentials
- ❌ Never use the root account for daily operations

---

## Common IAM Exam/Interview Points

- IAM is **global** — not region-specific
- Policies are evaluated across all applicable sources (identity-based + resource-based)
- A **Permission Boundary** restricts what a user/role *can* be allowed — it doesn't grant permissions on its own
- **STS (Security Token Service)** issues temporary credentials when assuming a role
- The `arn:aws:iam::aws:policy/AdministratorAccess` policy grants full access to all AWS services

---

## ARN Format for IAM

```
arn:aws:iam::<account-id>:<resource-type>/<resource-name>

# Examples
arn:aws:iam::123456789012:user/alice
arn:aws:iam::123456789012:group/developers
arn:aws:iam::123456789012:role/EC2S3ReadRole
arn:aws:iam::aws:policy/ReadOnlyAccess
```

---

*Last updated: May 2026*