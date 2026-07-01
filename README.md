# Cloud Security with AWS IAM

**AWS Services Used:** IAM, EC2  
**Difficulty:** Beginner  


---

## Overview

This project demonstrates how to use AWS Identity and Access Management (IAM) to implement cloud security best practices. The goal was to simulate a real-world scenario where a development team has scoped, least-privilege access to only the AWS resources they need — and nothing more.

Key security concepts applied:
- IAM users and group-based access control
- Custom IAM policies with tag-based conditions
- Explicit deny statements to protect resource integrity
- Account alias configuration for secure, human-readable sign-in URLs
- Policy simulation and live permission testing

---

## Architecture

```
AWS Account (aws-lucious)
│
├── IAM User Group: work-dev-group
│   └── Attached Policy: WorkdevEnvironmentPolicy
│       ├── ALLOW: All EC2 actions on resources tagged Env=development
│       ├── ALLOW: EC2 Describe* on all resources (read-only visibility)
│       └── DENY:  ec2:DeleteTags, ec2:CreateTags (tag protection)
│
├── IAM User: work-dev-luc (member of work-dev-group)
│
└── EC2 Instances
    ├── nextwork-prod-luc  (production — no dev tag, access restricted)
    └── next-dev-luc       (Env=development tag — accessible by dev user)
```

---

## What I Built

### 1. Account Alias
Created a custom account alias (`aws-lucious`) to replace the default numeric AWS account ID in the sign-in URL, making it easier for IAM users to log in securely:
```
https://aws-lucious.signin.aws.amazon.com/console
```

### 2. IAM User and Group
- Created IAM user **work-dev-luc** to represent a developer on the team
- Created IAM user group **work-dev-group** to manage permissions at the group level rather than per-user, following AWS best practices

### 3. Custom IAM Policy (WorkdevEnvironmentPolicy)
Wrote a customer-managed IAM policy in JSON with three statements:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Env": "development"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": ["ec2:DeleteTags", "ec2:CreateTags"],
      "Resource": "*"
    }
  ]
}
```

**Why this policy design matters:**
- The first statement uses a **tag-based condition** to scope access — the developer can only interact with EC2 instances tagged `Env=development`, preventing any accidental or unauthorized actions on production resources
- The second statement allows read-only `Describe` access across all resources so the developer can view the environment without being able to modify it
- The third statement explicitly **denies tag modification** — without this, a developer could tag a production instance as `development` and bypass the condition entirely

### 4. EC2 Instances with Environment Tags
Launched two EC2 instances to simulate a production/development split:

| Instance Name | Tag | Access Level |
|---------------|-----|-------------|
| nextwork-prod-luc | (none) | Restricted — no dev tag |
| next-dev-luc | Env=development | Accessible by work-dev-luc |

### 5. Permission Testing

**IAM Policy Simulator:**  
Used the IAM Policy Simulator to validate the policy before live testing:
- `StopInstances` on a resource tagged `Env=development` → **Allowed**
- `StopInstances` without the environment tag → **Denied (implicit)**
- `DeleteTags` → **Denied (explicit)**

**Live Console Test:**  
Logged in as `work-dev-luc` and attempted to stop the production instance (`nextwork-prod-luc`). AWS returned an access denied error confirming the policy was working as intended:

> *"User: arn:aws:iam::252624323229:user/work-dev-luc is not authorized to perform: ec2:StopInstances"*

The restricted user also had access denied to Security Hub and Service Catalog, confirming that least privilege was enforced across the account.

---

## Key Takeaways

- **Least privilege is enforced at the policy level, not just by role assignment.** Writing granular JSON policies with conditions is more secure than attaching broad AWS managed policies.
- **Tag-based access control is a powerful pattern** for environment separation (dev vs. prod) without needing separate AWS accounts.
- **Explicit Deny always wins.** The `ec2:DeleteTags` deny statement prevents privilege escalation through tag manipulation, which is a real attack vector.
- **The IAM Policy Simulator is essential** for validating policies before deploying them — it saves time and prevents accidental misconfigurations.
- **Group-based IAM management scales better** than assigning policies directly to individual users.

---

## Skills Demonstrated

`AWS IAM` · `EC2` · `Least Privilege` · `Tag-Based Access Control` · `Customer Managed Policies` · `IAM Policy Simulator` · `Cloud Security` · `JSON Policy Writing`

---

## Connect

**GitHub:** [github.com/lou7776](https://github.com/lou7776)  
**LinkedIn:linkedin.com/in/lucious-lokko
