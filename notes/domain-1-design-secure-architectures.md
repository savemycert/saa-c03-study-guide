# Domain 1: Design Secure Architectures (30%)

This is the largest SAA-C03 domain. It covers who can access AWS resources and through which mechanism, how to secure the network and application layers of a workload, and how to protect data with encryption, access controls, and retention. Most questions describe a working setup and ask for the option that is MOST secure, has the LEAST operational overhead, or follows least privilege.

## 1.1 Design secure access to AWS resources

- **Baseline hardening:** MFA on root, no root access keys, root only for root-only tasks, and permissions granted to groups rather than users. "Admins sign in as root" is the flaw to fix.
- **Least privilege:** scope to specific actions, ARNs, and conditions; `AdministratorAccess` or `"Resource": "*"` loses to a tighter option.
- **Policy evaluation, in three rules:**
  - Everything starts as an implicit deny.
  - An explicit deny beats every allow.
  - Guardrail layers (SCPs, permissions boundaries, session policies) intersect, so an action must pass all of them.
- Within one account, identity-based and resource-based policies combine as a **union**: an allow in either one is enough if nothing denies.
- **Debug cue:** "allowed but still denied" means an explicit deny in an SCP, boundary, or resource policy.

| Actor needing access | Mechanism |
|---|---|
| Application on EC2, Lambda, or ECS | IAM role (instance profile, execution role, task role) |
| Employees across many accounts, with an existing AD or IdP | IAM Identity Center with permission sets, federated to that source |
| Principal in another account, broad or multi-service access | Cross-account role, assumed with `sts:AssumeRole` |
| Principal in another account that needs one resource and must keep its own permissions | Resource-based policy on the target (bucket, queue, key, function) |
| Third-party SaaS vendor | Cross-account role whose trust policy requires an **external ID** |
| Customers of your web or mobile app | Amazon Cognito |

- **Roles over keys:** if something can use a role, it should. STS credentials expire on their own, so there is nothing to rotate or leak. Rotating access keys more often never beats removing them.
- **Role switching:** a modest daily identity plus an assumable privileged role (optionally MFA-gated), logged in CloudTrail.
- **Cross-account nuance:** assuming a role *replaces* your permissions for that session, while a resource-based policy *adds* access on top of your own. A Lambda function that reads a bucket in account B and writes to a table in account A in the same run should use a bucket policy in B, not a role in B.
- **External ID** defends against the confused deputy. A vendor asking for access keys is the anti-pattern.
- **Federation:** per-account SAML trust (`AssumeRoleWithSAML`) works, but Identity Center is the lower-overhead answer.
- **AWS Organizations:** a management account plus member accounts, grouped into OUs designed around policy needs (Production, Sandbox, Security) rather than the org chart.
- **SCP facts:**
  - An SCP never grants anything. It only caps the maximum permissions.
  - SCPs inherit down through OUs.
  - They constrain member-account **root users**.
  - They have **no effect on the management account**.
- **SCP cue:** "no one in any account, including administrators, may..." Admins can edit IAM policies and boundaries in their own account, but not an SCP set above it.
- **Permissions boundary:** a cap on a single user or role. It is the answer to "let developers create roles without escalating privileges."
- **AWS Control Tower:** a managed landing zone built on Organizations that includes log archive and audit accounts.
  - **Preventive controls** are implemented as SCPs and block the action.
  - **Detective controls** are implemented as AWS Config rules and flag the resource after the fact.
  - **Account Factory** provisions new accounts that are compliant from the start.
  - Choose it for a "governed multi-account setup with least effort". It complements Organizations.

📖 Full lesson: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 1.2 Design secure workloads and applications

- **The route table decides the subnet type:**

| Subnet | Default route | Belongs here |
|---|---|---|
| Public | Internet gateway | Internet-facing ALB, NAT gateways |
| Private | NAT gateway in a public subnet | App servers (outbound-only internet) |
| Isolated | No internet gateway or NAT route | Databases that only talk to the app tier |

- **Minimal exposure:** "app instances with public IPs" or "RDS in a public subnet" is the flaw. Move the tier inward.
- **NAT for HA:** one NAT gateway per AZ, so an AZ failure does not cut outbound access elsewhere.
- **Security groups vs NACLs:**
  - A security group is stateful, attaches to an ENI, has allow rules only, and evaluates all rules together.
  - A NACL is stateless, attaches to a subnet, has allow and deny rules, and uses the first match in rule-number order.
  - Blocking a specific IP or CIDR range needs a NACL.
  - If requests arrive but replies never return, suspect a NACL that is missing outbound or ephemeral-port rules.
- **Reference security groups by ID:** chain ALB SG → app SG → DB SG instead of using CIDRs. Membership then follows Auto Scaling without any maintenance.
- **VPC endpoints:**
  - A **gateway endpoint** is a route-table target, exists only for S3 and DynamoDB, and has no charge.
  - An **interface endpoint** (PrivateLink) is an ENI with a security group, covers most other services, and is billed.
  - For S3 or DynamoDB from inside the VPC with a cost qualifier, choose the gateway endpoint.
  - For S3 from on premises over VPN or Direct Connect, choose an interface endpoint, because a gateway endpoint only serves its own VPC.
  - An endpoint policy plus a bucket policy that allows access only through that endpoint makes S3 fully private.
- **Admin access:** SSM Session Manager beats a bastion host: no inbound ports, no SSH keys, IAM-authenticated and logged.
- **Cognito:**
  - A **user pool** handles authentication (sign-up, sign-in, MFA, social or SAML/OIDC login) and issues JWTs. It can act as an API Gateway or ALB authorizer.
  - An **identity pool** exchanges a token for temporary AWS credentials under an IAM role, and also supports guest identities.
  - "Mobile app uploads to S3" means an identity pool. Never ship access keys inside an app.

| Threat or requirement | Service |
|---|---|
| Layer 3/4 DDoS (SYN flood, UDP reflection) | Shield Standard (automatic); Shield Advanced for a response team and cost protection |
| SQL injection, XSS, rate abuse | AWS WAF on CloudFront, ALB, API Gateway, or AppSync |
| Compromised instance, crypto-mining, suspicious API calls | GuardDuty (reads CloudTrail, VPC Flow Logs, DNS logs) |
| PII sitting in S3 | Macie |
| Credentials hard-coded in code | IAM roles plus Secrets Manager or Parameter Store |

- **Layer traps:** WAF for a SYN flood, GuardDuty for PII, Macie for intrusions. GuardDuty detects; it does not block.
- **Secrets:**
  - "Automatic rotation" or cross-account secret sharing means Secrets Manager.
  - "Most cost-effective" storage of config or static secrets means a Parameter Store SecureString, which has no built-in rotation.
  - Both stores encrypt with KMS, so "more secure" alone is not a reason to pick Secrets Manager.
- **Hybrid links:**
  - Site-to-Site VPN uses IPsec over the internet. It is fast to set up but has variable performance.
  - Direct Connect is private and consistent but **not encrypted by default**.
  - "Encrypted and consistent" means a VPN over Direct Connect. A VPN is also the cheaper failover path for a Direct Connect link.

📖 Full lesson: [Securing Workloads: VPC Design, Security Groups vs NACLs, WAF and Shield](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-vpc-security-secure-workloads/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 1.3 Determine appropriate data security controls

- **KMS key types:**
  - **AWS owned** keys are invisible to you and not auditable per account.
  - **AWS managed** keys (`aws/s3`, `aws/ebs`) appear in your account and rotate yearly. You cannot edit their key policy, disable them, or use them cross-account.
  - **Customer managed** keys are required whenever the stem mentions controlling who can use the key, rotation choices, disabling the key, or sharing with another account.
- **Key policy** is the final authority over a key; even admins are locked out if it does not allow them. Grants let services such as EBS use the key.
- **Envelope encryption:** KMS wraps a data key that encrypts the data locally, so disabling one KMS key cuts off everything it protects.
- **KMS vs CloudHSM:** choose CloudHSM only for "single-tenant", "dedicated hardware", or "AWS must never access key material". It loses any least-overhead question. A **KMS custom key store** backed by CloudHSM gives you both native service integration and dedicated HSMs.

| S3 option | Choose when |
|---|---|
| SSE-S3 (default) | Only "encrypt at rest" is required |
| SSE-KMS, customer managed key | Audit key usage, control who can decrypt, cross-account access |
| SSE-KMS + Bucket Keys | KMS request volume is driving cost |
| SSE-C | You supply the key on every request and S3 does the encryption |
| Client-side | AWS must never see plaintext |

- **KMS permission trap:** "has `s3:GetObject` but gets access denied on some objects" means `kms:Decrypt` is missing on the key.
- **Mandate encryption:** a bucket policy denying `PutObject` without the required encryption header.
- **EBS and RDS encryption is fixed at creation.** To encrypt an existing resource: snapshot → copy the snapshot with encryption on (choose the key) → new volume or restored instance → cut over. The same path re-keys data. Any option that enables encryption "in place" is wrong.
  - EBS encryption by default covers all future volumes with the least effort.
  - Snapshots encrypted with the default AWS managed key cannot be shared, so sharing needs a customer managed key.
  - KMS keys are regional, so a cross-Region copy needs a key in the destination Region.
- **TLS and ACM:**
  - ACM public certificates cannot be exported to EC2. Use ACM Private CA or your own certificates for instances.
  - ACM certificates are regional, **except that CloudFront needs its certificate in us-east-1**.
- **Enforcing HTTPS:**
  - ALB: redirect the HTTP listener to HTTPS.
  - CloudFront: set the viewer protocol policy to redirect or HTTPS-only, and set the origin protocol policy for the origin leg.
  - S3: deny requests where `aws:SecureTransport` is false.
- **Rotation:**
  - Customer managed keys can opt in to automatic rotation. Old key material is kept, the key ID and alias do not change, and no data is re-encrypted.
  - Keys with imported material or in a custom key store must be rotated manually: create a new key and repoint the alias.
  - ACM renews certificates it issued with DNS validation. **Imported certificates are never auto-renewed.**
- **Access layers:**
  - A bucket policy handles cross-account access, bucket-wide guardrails, and conditions such as `aws:sourceVpce`.
  - **Block Public Access** at the account level is the preventive answer to "no bucket may ever be public".
  - "Must not traverse the internet" calls for a VPC endpoint, not encryption.
- **Retention mapping:**

| Requirement wording | Control |
|---|---|
| No one, including root, may delete (regulatory) | Object Lock **compliance** mode (needs versioning) |
| Protected, but admins need an override | Object Lock governance mode |
| Unknown end date (litigation) | Legal hold |
| Recover from overwrites or deletes | Versioning (optionally MFA Delete) or backups |
| Cheaper storage as data ages | Lifecycle rules to IA or Glacier classes |
| Copy in another Region or account | Cross-Region Replication (versioning on both buckets) |
| Point-in-time recovery across services | AWS Backup; Vault Lock makes backups immutable |

- **Replication vs backup:** replication copies deletes and corruption too; backups give points in time. Seven-year retention usually pairs compliance-mode Object Lock with a lifecycle rule to Glacier.

📖 Full lesson: [AWS Data Security Controls: KMS Encryption, S3 Encryption, and Key Management](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-data-security-encryption-kms/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

[← Back to the study guide](../README.md)
