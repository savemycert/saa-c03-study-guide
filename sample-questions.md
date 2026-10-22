# SAA-C03 Sample Questions with Answers

20 worked practice questions for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [SAA-C03 practice questions](https://www.savemycert.com/practice/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide) on SaveMyCert.

## Question 1

*Design Secure Architectures*

**A company manages 40 AWS accounts in AWS Organizations, grouped into a Development OU and a Production OU. The security team requires that no user in a Development account, including account administrators who hold full IAM permissions, can stop or delete AWS CloudTrail trails. Which approach enforces this requirement?** Choose one.

- **a.** Attach an IAM policy that denies cloudtrail:StopLogging and cloudtrail:DeleteTrail to the IAM groups and roles in each Development account.
- **b.** Attach a service control policy that denies cloudtrail:StopLogging and cloudtrail:DeleteTrail to the Development OU in the organization.
- **c.** Deploy an AWS Config rule to each Development account that detects a stopped or deleted CloudTrail trail and alerts the security team.
- **d.** Attach a permissions boundary that denies cloudtrail:StopLogging and cloudtrail:DeleteTrail to the IAM roles in each Development account.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Account administrators with full IAM permissions can detach or edit an IAM policy deployed inside their own account, so this control does not bind them.
- **b** ✅ An SCP is managed from the organization, above the member accounts. It is inherited by every account in the OU and restricts every principal there, including administrators and the member account root user, and no one inside those accounts can detach it.
- **c** ❌ A Config rule is a detective control. It reports that logging was stopped after it happened instead of preventing the action, which the requirement demands.
- **d** ❌ A permissions boundary is an IAM object managed inside the account. Administrators who control IAM can remove or replace it, and it does not apply to the member account root user.

**The concept.** A restriction that must bind account administrators cannot live inside the accounts those administrators control. Service control policies are applied from AWS Organizations and set the maximum permissions for every principal in the member accounts they reach.

**Why this is correct.** The SCP on the Development OU works because it is enforced at a layer no Development-account principal can change: it is inherited by each account in the OU, restricts member-account root users too, and denies the CloudTrail actions whatever the IAM policies allow. The IAM deny policy and the permissions boundary deny the same actions, but both are IAM objects inside the account, so an administrator with full IAM permissions can detach them. The Config rule is detective: it tells the security team that logging stopped instead of stopping the action.

**How to reason it out**

1. Classify the requirement: it is preventive and must bind administrators of the affected accounts.
2. Ask who controls each option: IAM policies and permissions boundaries are managed by the account's own administrators.
3. Discard detective controls such as Config rules when the requirement is to prevent an action.
4. Place the deny above the accounts: an SCP attached to the Development OU is inherited by every account in it.

> **Exam tip:** "No one in these accounts, including admins" means an SCP on the OU; anything managed inside the account can be removed by its admins.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 2

*Design Secure Architectures*

**A solutions architect creates a new member account in AWS Organizations and attaches a service control policy to it that explicitly allows s3:* and ec2:*. A newly created IAM user in the account has no IAM policies attached. When the user attempts to list S3 buckets, the request fails with an access denied error. What explains this behavior?** Choose one.

- **a.** An SCP attached directly to an account is ignored until the same policy is attached to the organizational unit that contains it.
- **b.** SCP changes take up to 24 hours to reach a newly created member account, so the S3 allow is not yet in effect for its IAM users.
- **c.** Each S3 bucket needs a bucket policy that names the IAM user before an IAM identity in the account can list the buckets.
- **d.** An SCP only filters what IAM policies allow, so the user still needs an identity-based policy that allows the S3 actions.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ SCPs can be attached to the organization root, to OUs or directly to accounts, and an SCP attached to an account takes effect there.
- **b** ❌ SCP changes take effect within minutes, not a day. Waiting would not help either, because the SCP does not grant the user anything.
- **c** ❌ Listing buckets (s3:ListAllMyBuckets) is an account-level action authorized by an identity-based policy; bucket policies are not required for same-account access.
- **d** ✅ Correct. An SCP sets the maximum permissions available in the account but grants nothing. With no identity-based policy, the user has no allow, so the request is implicitly denied.

**The concept.** Service control policies are guardrails: they define the maximum permissions available to principals in member accounts, but they never grant permissions. An action succeeds only if an identity-based or resource-based policy allows it and every applicable SCP permits it.

**Why this is correct.** The user has no identity-based policy, so nothing grants the S3 action and the request is implicitly denied. The SCP's allow statements only mean that S3 and EC2 are not filtered out; they do not give anyone access. The propagation explanation is wrong because SCP changes apply within minutes and would not grant access anyway. SCPs attached directly to accounts are valid. Bucket policies are not needed for a same-account principal to list buckets; an identity-based policy is.

**How to reason it out**

1. Note that the user has no IAM policies at all.
2. Recall that an SCP sets a ceiling and never grants a permission.
3. With no allow anywhere, the default implicit deny applies.
4. Fix: attach an identity-based policy (preferably to a group) that allows the needed S3 actions.

> **Exam tip:** An SCP allow is permission to be granted, not a grant; someone still has to write an IAM policy.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 3

*Design Secure Architectures*

**A security team attaches a service control policy to the root of its organization that denies all actions in AWS Regions other than eu-west-1 and eu-central-1. One month later, a review finds new EC2 instances running in us-east-1 that administrators launched after the SCP took effect. All of the instances are in a single account. What is the MOST likely explanation?** Choose one.

- **a.** The administrators used a role whose permissions boundary allows us-east-1, which takes precedence over the SCP for that role.
- **b.** The administrators launched the instances as the account's root user, and SCPs do not restrict the root user of an account.
- **c.** SCPs attached to the organization root are inherited by OUs but not by accounts placed directly under the organization root.
- **d.** The account that runs the instances is the organization's management account, which the Region-deny SCP does not restrict.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ A permissions boundary can only narrow permissions further. It cannot override an SCP deny; the effective permissions are the intersection of every layer.
- **b** ❌ SCPs do restrict the root user of member accounts. Signing in as root would not bypass the Region deny in a member account.
- **c** ❌ An SCP attached to the organization root applies to every member account, whether the account sits in an OU or directly under the root.
- **d** ✅ Correct. SCPs have no effect on the management account, including its root user, so the Region deny never applied there. All instances being in one account points to this.

**The concept.** SCPs apply to member accounts only. The management account is outside their reach, so it is the one account where an organization-wide guardrail does not hold. Inside member accounts, SCPs restrict every principal, including the root user.

**Why this is correct.** Only one account is affected, and the deny was attached at the organization root, so the guardrail failed in exactly one place: the management account, which SCPs never affect. The root-user explanation is the tempting inversion of the rule; SCPs do bind member-account root users. Accounts directly under the root inherit root-level SCPs like any other account. A permissions boundary cannot loosen an SCP because every guardrail layer must permit an action for it to succeed.

**How to reason it out**

1. Note the clue: the policy failed in a single account, not across the organization.
2. Recall the SCP scope: all member accounts, including their root users, but not the management account.
3. Rule out boundary or root-user bypasses: layers intersect, and member root users are bound.
4. Conclude the workloads ran in the management account, and keep workloads out of it.

> **Exam tip:** SCPs bind member-account root users but never the management account, so keep workloads out of the management account.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 4

*Design Secure Architectures*

**A company runs a two-tier web application in a single VPC. The web servers and an RDS MySQL database both run in a public subnet, and the database's security group allows inbound traffic on port 3306 from 0.0.0.0/0 so that the application can always reach it. Security now requires that the database be unreachable from the internet. Which redesign is MOST secure?** Choose one.

- **a.** Move the database to a private subnet with no route to an internet gateway or NAT gateway, and allow inbound 3306 from the web tier's security group.
- **b.** Keep the database in the public subnet and narrow its security group so inbound 3306 is allowed from the web servers' public IP addresses.
- **c.** Move the database to a private subnet whose route table sends 0.0.0.0/0 to a NAT gateway, and allow inbound 3306 from the VPC's CIDR block.
- **d.** Keep the database in the public subnet and add network ACL rules that deny inbound 3306 from the internet except from the web servers' IP addresses.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The isolated subnet removes any internet path at the routing layer, and the security-group reference restricts access to the web tier by relationship rather than by IP.
- **b** ❌ The database stays in an internet-routable subnet, the rule breaks whenever web-server addresses change, and one security group mistake re-exposes it to the internet.
- **c** ❌ The database is no longer reachable from the internet, but the NAT route still gives it an outbound internet path, and the VPC CIDR rule admits every resource in the VPC rather than only the web tier.
- **d** ❌ A stateless, IP-based filter layered over the same public placement narrows exposure without removing the internet route, and its address list goes stale as web servers scale.

**The concept.** A subnet's route table defines its exposure: no internet gateway route and no NAT route means nothing in the subnet can reach or be reached from the internet. Combined with a security group that references the calling tier's group by ID, this removes exposure at the design level rather than patching it.

**Why this is correct.** Moving the database into a subnet with no internet gateway or NAT route removes internet reachability structurally, and the security-group reference lets any instance carrying the web tier's group connect while nothing else can, with no rule maintenance as the tier scales. The NAT-routed private subnet is the close miss: it does stop inbound internet connections, but it leaves the database an outbound internet path it has no need for, and a VPC-wide CIDR rule admits far more than the web tier. Narrowing the security group to web-server IPs reduces exposure without removing it: placement is still public and one rule edit re-exposes it. The NACL rules layer a stateless, IP-based filter over the same flawed placement.

**How to reason it out**

1. Identify both flaws: internet-routable subnet placement and a 0.0.0.0/0 database rule.
2. Fix placement first: put the database in a subnet whose route table has no IGW or NAT entry.
3. Fix access next: allow the database port only from the web tier's security group, referenced by ID.
4. Prefer structural removal of exposure over options that merely narrow it (CIDR rules, NACLs).
5. Reject a private placement that still carries a NAT route or a VPC-wide source: the data tier needs neither.

> **Exam tip:** Databases belong in isolated subnets with inbound allowed only from the application tier's security group by ID — never in public subnets with tightened IP rules.

</details>

📖 Learn this topic: [Securing Workloads: VPC Design, Security Groups vs NACLs, WAF and Shield](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-vpc-security-secure-workloads/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 5

*Design Secure Architectures*

**A financial services company stores compliance reports in Amazon S3 and runs its reporting servers on EBS-backed EC2 instances. The security team must decide exactly which IAM principals may use the encryption key, and encrypted EBS snapshots of the servers must be shared with an external auditor's AWS account. Which type of encryption key meets these requirements?** Choose one.

- **a.** An AWS managed KMS key that each service creates in the company's account
- **b.** An Amazon S3 managed key applied through SSE-S3 default encryption on the bucket
- **c.** A customer managed KMS key that the company creates and administers itself
- **d.** An AWS owned KMS key that each service holds outside the company's account

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ AWS managed keys such as aws/s3 and aws/ebs are visible in the account, but their key policies cannot be edited and snapshots encrypted with them cannot be shared with another account.
- **b** ❌ SSE-S3 keys are managed entirely by S3 with no key policy, and they do not encrypt EBS snapshots at all, so neither requirement is met.
- **c** ✅ The company writes this key's policy, so it decides which principals may use it, and the policy can allow the auditor's account to use the key for the shared snapshots.
- **d** ❌ AWS owned keys are never visible to the customer: there is no key policy to write and no way to let another account use them.

**The concept.** KMS has three ownership levels: AWS owned keys (invisible, no control), AWS managed keys (visible, but AWS sets the key policy and they cannot be used across accounts) and customer managed keys (you write the key policy and can grant other accounts use of the key).

**Why this is correct.** Two requirements decide it: the security team must control which principals can use the key, and encrypted snapshots must reach another account. Both need an editable key policy, which only a customer managed key has. AWS managed keys are the tempting choice because they encrypt S3 and EBS with no effort, but their policies are fixed by AWS and EBS refuses to share snapshots encrypted with them. AWS owned keys and SSE-S3 keys offer even less control, and SSE-S3 does not apply to EBS.

**How to reason it out**

1. Pick out the control requirements: choose who may use the key, and share encrypted snapshots with another account.
2. Both need a key policy you can write.
3. Eliminate AWS owned keys and SSE-S3 keys: no key policy is exposed to you.
4. Eliminate AWS managed keys: their key policy is fixed and they cannot be used for cross-account snapshot sharing.
5. Choose a customer managed key.

> **Exam tip:** Edit the key policy, disable the key, or share encrypted data with another account: that is a customer managed KMS key.

</details>

📖 Learn this topic: [AWS Data Security Controls: KMS Encryption, S3 Encryption, and Key Management](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-data-security-encryption-kms/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 6

*Design Secure Architectures*

**A financial services company stores compliance logs in an S3 bucket in a dedicated audit account within its AWS organization. Regulators require assurance that no principal that runs the company's workloads, including member-account root users, can delete the bucket. A platform team also proposes running a new log-processing workload in the organization's management account. Which combination of actions should a solutions architect take? (Select TWO.)** Choose 2.

- **a.** Apply a permissions boundary that denies s3:DeleteBucket on the audit bucket to each member account's root user.
- **b.** Attach an SCP that denies s3:DeleteBucket on the audit bucket to the OU that contains the audit account only.
- **c.** Attach an SCP that denies s3:DeleteBucket on the audit bucket to the OUs that contain every member account.
- **d.** Attach the same SCP to the organization's management account so the proposed workload there is restricted too.
- **e.** Run the proposed log-processing workload in a member account rather than in the organization's management account.

<details>
<summary>Show answer and explanation</summary>

**Answer: c, e**

- **a** ❌ Permissions boundaries attach to IAM users and roles only, not to the root user, and account administrators could remove them.
- **b** ❌ This binds principals in the audit account, but principals in other member accounts that a bucket policy lets in would sit outside the guardrail, so the assurance is incomplete.
- **c** ✅ Correct. An SCP on the OUs is inherited by each member account and restricts every principal there, including the root user, and member-account administrators cannot detach it.
- **d** ❌ An SCP has no effect on the management account, so the workload there would remain unrestricted. Moving the workload to a member account is what closes the gap.
- **e** ✅ Correct. SCPs do not apply to the management account, so a workload there would sit outside the guardrail. Running it in a member account keeps it covered.

**The concept.** Two SCP scope rules combine here: SCPs bind every principal in member accounts, including root users, and SCPs do not affect the management account at all.

**Why this is correct.** The deny SCP on the OUs that contain every member account blocks the deletion for every principal in those accounts, root included, and is inherited by accounts added to those OUs later. Attaching it only to the audit account's OU is the near-twin: it covers the bucket's own account but not principals in other member accounts. The proposed management-account workload is the second risk. Attaching the SCP to the management account looks like a fix but does nothing, because SCPs never apply there, so the workload must run in a member account. Permissions boundaries cannot be attached to the root user, and account administrators could remove them anyway.

**How to reason it out**

1. Identify who must be restricted: every workload principal in member accounts, including root users.
2. Choose the layer that binds root users from above: an SCP on the OUs that contain every member account, not just the bucket's account.
3. Check the coverage gap: SCPs do not reach the management account.
4. Close it by running the proposed workload in a member account.

> **Exam tip:** SCPs cover member accounts and their root users; the management account is outside every SCP, so no workloads there.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 7

*Design Resilient Architectures*

**A photo-sharing startup runs a web tier that synchronously calls a fleet of EC2 instances to generate image renditions after each upload. When a post goes viral, the rendition fleet saturates, upload requests time out, and some uploads are never processed. The company needs a redesign that loses no rendition job and handles unpredictable spikes as cost-effectively as possible. What should a solutions architect recommend?** Choose one.

- **a.** Place an Amazon SQS queue between the web tier and the rendition fleet, and scale the fleet's Auto Scaling group on the queue backlog.
- **b.** Publish each upload job to an Amazon SNS topic, and subscribe the rendition instances in an Auto Scaling group to it as HTTP endpoints.
- **c.** Run the rendition fleet in an Auto Scaling group whose minimum capacity matches the largest viral spike seen so far.
- **d.** Put an Application Load Balancer in front of the rendition fleet, scale the fleet on CPU utilization, and raise the web tier's request timeout.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The queue holds each job durably until a worker deletes it after success, so nothing is lost, and the fleet grows with the backlog during a spike and shrinks afterward, so the company pays only for the capacity it uses.
- **b** ❌ SNS pushes at its own pace rather than letting saturated workers pull at theirs; once delivery retries to overwhelmed endpoints are exhausted the job is gone, so it is not a durable consumer-paced buffer.
- **c** ❌ Sizing the floor for the worst spike so far keeps the synchronous call path, still drops jobs when a bigger spike arrives, and pays for peak capacity through every quiet hour.
- **d** ❌ This is still a synchronous chain: while new instances launch, saturated targets keep timing out, and the load balancer stores nothing, so jobs caught in that window are still lost.

**The concept.** Inserting a queue between tiers converts a traffic spike into a durable backlog that a worker tier drains at its own pace. The tiers are decoupled, so each scales on its own signal and no work is lost while consumers catch up.

**Why this is correct.** The SQS design meets both requirements: SQS keeps every job until a worker processes and deletes it (no lost renditions), and scaling the worker group on queue backlog adds capacity only while a backlog exists (cost-effective for unpredictable spikes). The load balancer plus CPU scaling option is the tempting near-twin: it does scale, but the web tier still waits on the rendition fleet, and anything that arrives while the fleet is saturated and new instances are still booting times out with nowhere to wait. Sizing the minimum capacity for the biggest spike so far keeps the same fragile call path, has a ceiling the next viral post can exceed, and bills idle capacity around the clock. SNS looks like decoupling but is push delivery without consumer pacing; overwhelmed HTTP subscribers exhaust retries and the job is dropped.

**How to reason it out**

1. Diagnose the failure: synchronous calls into a saturated tier lose work during spikes, which is tight coupling.
2. Add a durable intermediary: an SQS queue stores each job until a worker finishes and deletes it.
3. Scale the consumer on the right signal: queue backlog grows the worker group during a spike and shrinks it afterward.
4. Check cost: elastic workers that follow the backlog beat a permanently peak-sized floor.

> **Exam tip:** When spikes lose work between tiers, buffer with SQS and scale the workers on queue backlog; a load balancer spreads requests but stores none.

</details>

📖 Learn this topic: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 8

*Design Resilient Architectures*

**An application uses an SQS standard queue between its API tier and a worker fleet of six EC2 instances. During ingest bursts the backlog grows to hundreds of thousands of messages and takes hours to drain, yet the fleet sits mostly idle between bursts. The bursts do not occur at predictable times. Which solution drains bursts quickly with the LEAST operational overhead?** Choose one.

- **a.** Move the workers into an Auto Scaling group with a target tracking policy on the queue's backlog per instance.
- **b.** Convert the standard queue to a FIFO queue with high-throughput mode so that the six workers receive messages in larger batches.
- **c.** Replace the six workers with larger instance types and enable long polling to cut empty receives while the backlog drains.
- **d.** Move the workers into an Auto Scaling group with scheduled actions that add instances during the hours when last month's bursts occurred.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Backlog per instance (visible messages divided by running workers) tracks exactly how far behind the fleet is, and target tracking adds and removes workers to hold it at the acceptable level with no further tuning.
- **b** ❌ FIFO adds ordering and deduplication, not consumer capacity; six workers process the same backlog at the same rate, and FIFO's lower throughput ceiling is a step in the wrong direction.
- **c** ❌ Bigger instances raise a fixed ceiling and cost more during the idle periods, and long polling reduces empty responses and request cost, not processing time.
- **d** ❌ Scheduled scaling suits known, recurring cycles. Bursts that arrive at unpredictable times fall outside the schedule, and the schedule needs constant re-tuning.

**The concept.** A worker tier behind a queue should scale on how far behind it is. AWS recommends a target tracking policy on backlog per instance: visible messages divided by running instances, compared with the backlog one instance can clear in the acceptable latency.

**Why this is correct.** Target tracking on backlog per instance turns the queue into the scaling signal: when a burst lands the metric jumps, the group adds workers until each one's share of the backlog is back at the target, and it scales in when the queue empties. That fits unpredictable bursts with nothing to operate afterward. Scheduled scaling is the trap for 'bursts happen sometimes': it only helps when the timing is known. Switching to FIFO changes delivery semantics, not throughput, and actually lowers the ceiling. Larger instances with long polling is vertical scaling plus a cost optimization; it keeps a fixed fleet that is too small during bursts and wasteful between them.

**How to reason it out**

1. Identify the signal that measures the problem: the queue backlog, not CPU or the clock.
2. Rule out scheduled scaling because the bursts are unpredictable.
3. Rule out queue-type and instance-size changes because neither adds consumers when the backlog grows.
4. Pick target tracking on backlog per instance, which scales out and in with no manual tuning.

> **Exam tip:** Queue-fed workers scale on backlog per instance with target tracking; schedules fit only predictable cycles.

</details>

📖 Learn this topic: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 9

*Design Resilient Architectures*

**A company runs a three-tier application in a single Region and must be able to recover it in a second Region after a regional outage. The business has set an RTO of 60 minutes and an RPO of 5 minutes, and has directed the architecture team to choose the cheapest disaster recovery strategy that meets both objectives. Which strategy should the team select?** Choose one.

- **a.** Run the full stack at production capacity in both Regions on replicated data, with Route 53 latency-based routing serving users from both Regions.
- **b.** Run a scaled-down but fully working copy of the whole stack in the second Region on replicated data and scale it out during a failover.
- **c.** Replicate the database continuously to the second Region and keep AMIs and launch templates there, starting the application tier only on failover.
- **d.** Copy database snapshots and AMIs to the second Region every hour and rebuild the stack with CloudFormation after a disaster.

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Active-active meets the objectives by the widest margin but roughly doubles infrastructure cost, so it fails the cheapest-that-meets instruction most severely.
- **b** ❌ Warm standby meets both objectives, but it pays continuously for a running application tier that a 60-minute RTO does not need.
- **c** ✅ This is pilot light: continuous replication meets the 5-minute RPO, launching pre-staged compute fits a 60-minute RTO, and almost no compute runs while idle, so it is the cheapest strategy that meets both numbers.
- **d** ❌ Hourly snapshots allow up to an hour of data loss, which breaks the 5-minute RPO, and rebuilding the whole stack from templates is unlikely to finish inside 60 minutes.

**The concept.** DR strategy selection is a ladder of cost against recovery speed: extract the RTO and RPO first, then choose the cheapest strategy whose recovery characteristics satisfy both numbers.

**Why this is correct.** The two numbers do the elimination. An RPO of 5 minutes needs continuous replication, so any snapshot-based approach fails: even hourly snapshots can lose an hour of writes, and rebuilding the stack from templates usually runs past an hour as well. Pilot light, warm standby and active-active all meet the numbers: pilot light recovers in tens of minutes because the data is already live and only compute must launch, warm standby in minutes, active-active in seconds. The instruction to pick the cheapest strategy that meets both objectives then selects the lowest surviving rung, pilot light. Warm standby and active-active are over-delivery distractors: both work, both cost more, and a 60-minute RTO gives no reason to pay for either.

**How to reason it out**

1. Extract the objectives before reading options: RTO 60 minutes, RPO 5 minutes.
2. Eliminate snapshot-based recovery: its RPO equals snapshot frequency and the rebuild is slow.
3. Note that pilot light, warm standby and active-active all meet a 60-minute RTO with continuous replication.
4. Apply the cost instruction and choose the lowest rung that still meets both numbers: pilot light.

> **Exam tip:** Read the RTO and RPO first, then pick the cheapest DR rung that meets both; anything higher is over-delivery.

</details>

📖 Learn this topic: [AWS High Availability and Disaster Recovery: RTO, RPO, and the Four DR Levels](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-availability-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 10

*Design Resilient Architectures*

**A ticketing company's checkout API calls a seat-allocation service synchronously. During on-sale events for popular concerts the allocation service is overwhelmed, calls fail, and some paid checkouts never receive seats. Allocation may complete a few minutes after checkout, but no checkout can be dropped, and allocation capacity must grow with demand. Which combination of steps should a solutions architect take? (Select TWO.)** Choose 2.

- **a.** Have the checkout API retry failed allocation calls with exponential backoff and jitter before returning an error.
- **b.** Have the checkout API write each paid checkout to an Amazon SQS queue instead of calling the allocation service directly.
- **c.** Have the checkout API publish each paid checkout to an Amazon SNS topic with the allocation service subscribed over HTTPS.
- **d.** Run the allocation workers in an Auto Scaling group that scales on how much allocation work is waiting to be processed.
- **e.** Put an internal Application Load Balancer in front of the allocation service and scale its targets on request count.

<details>
<summary>Show answer and explanation</summary>

**Answer: b, d**

- **a** ❌ Backoff smooths retry storms, but a call that is still failing when the retries run out returns an error and the checkout is still lost.
- **b** ✅ The queue accepts every checkout instantly and keeps it until an allocation worker processes and deletes it, so an overwhelmed service delays allocation rather than losing it.
- **c** ❌ SNS pushes to the overwhelmed endpoint on its own schedule; after the delivery retries are exhausted the checkout is dropped, so it is not a durable buffer.
- **d** ✅ Scaling on the amount of pending allocation work (the queue backlog) adds workers while checkouts are waiting and removes them afterward, so capacity follows demand.
- **e** ❌ This scales the service but keeps the synchronous call path, so checkouts that arrive while targets are saturated or still launching fail with nowhere to wait.

**The concept.** Two halves make a spike-proof hand-off: a durable queue between producer and consumer so nothing is dropped, and a consumer tier that scales on the queue's backlog so capacity follows demand.

**Why this is correct.** The stem tolerates a few minutes of delay but no loss, which is exactly what an asynchronous queue provides: SQS stores each checkout until a worker deletes it, and scaling the worker group on the queue's backlog adds allocation capacity during an on-sale event. The load balancer with request-count scaling is the near-twin: it scales, but checkouts still depend on the allocation service answering in time, so the ones caught while it is saturated fail. Retrying with backoff and jitter is good client hygiene but only postpones the failure. SNS over HTTPS is push delivery; it has no consumer-paced storage, so an endpoint that stays overwhelmed past the retry policy loses the message.

**How to reason it out**

1. Read the constraint: delay is acceptable, loss is not, so the hand-off can be asynchronous.
2. Choose a durable buffer: SQS keeps each checkout until a worker processes and deletes it.
3. Pair it with consumer scaling on the queue backlog so capacity grows with demand.
4. Reject options that keep the synchronous call (load balancer, retries) or push without storage (SNS over HTTPS).

> **Exam tip:** No-loss plus elastic capacity = SQS between the tiers and workers scaling on queue backlog.

</details>

📖 Learn this topic: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 11

*Design Resilient Architectures*

**An internal document-archive application is used a few times per week. After a regional outage, the business can tolerate up to 24 hours of downtime and up to 24 hours of data loss. A consultant has proposed running a warm standby of the application in a second Region. What should a solutions architect recommend to minimize disaster recovery cost while still meeting the objectives?** Choose one.

- **a.** Change the proposal to backup and restore: copy daily backups to the second Region and rebuild the stack from templates.
- **b.** Change the proposal to a pilot light that continuously replicates the database to a live instance in the second Region.
- **c.** Keep the warm standby but shrink its application tier to a single small instance that is scaled out during a failover.
- **d.** Change the proposal to a Multi-AZ deployment in the current Region with automated daily snapshots kept in that Region.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Daily cross-Region backups give an RPO of at most a day and a template rebuild finishes in hours, so backup and restore meets both objectives with nothing running in the second Region.
- **b** ❌ Pilot light keeps a replicated database running around the clock to achieve an RPO of minutes; a 24-hour RPO does not justify that cost.
- **c** ❌ A smaller warm standby is still a running copy of the stack paying for compute every hour, which objectives of 24 hours do not require.
- **d** ❌ Multi-AZ and same-Region snapshots are cheap but stay inside the failed Region, so they cannot recover the application after a regional outage.

**The concept.** Backup and restore is the cheapest DR strategy and is the right choice whenever the business can tolerate an RTO and RPO measured in hours.

**Why this is correct.** Both objectives are 24 hours, which is backup-and-restore territory: daily backups copied to the second Region bound data loss at a day, and rebuilding the stack from infrastructure-as-code takes hours, well inside the RTO. Nothing runs in the recovery Region day to day, so the cost is only backup storage. A shrunken warm standby and a pilot light both work but keep resources running to buy recovery speed nobody asked for. The Multi-AZ option is the tempting cheap answer, but it protects only against failures inside the Region, so it fails the stated regional-outage requirement.

**How to reason it out**

1. Extract the objectives: RTO 24 hours, RPO 24 hours, recovery must happen in another Region.
2. Match hours-scale objectives to backup and restore with cross-Region copies and infrastructure as code.
3. Reject options that keep databases or compute running in the second Region, because they over-deliver.
4. Reject any option whose copies stay in the primary Region, because it cannot survive a regional outage.

> **Exam tip:** When RTO and RPO are measured in hours, cross-Region backup and restore is the cheapest strategy that still works.

</details>

📖 Learn this topic: [AWS High Availability and Disaster Recovery: RTO, RPO, and the Four DR Levels](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-availability-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 12

*Design High-Performing Architectures*

**A company is moving a self-managed relational database from on premises to a single EC2 instance. Monitoring shows a steady load of about 9,000 random IOPS at 200 MiB/s, and the application team has confirmed that single-digit-millisecond storage latency is acceptable. The data must persist across instance stops. Which EBS volume type meets these requirements MOST cost-effectively?** Choose one.

- **a.** General Purpose SSD (gp3) with IOPS and throughput raised above the baselines
- **b.** Throughput Optimized HDD (st1) sized to deliver the measured throughput
- **c.** Provisioned IOPS SSD (io2) with IOPS provisioned to cover the measured load
- **d.** NVMe instance store on a storage-optimized instance sized for the load

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ gp3 starts at 3,000 IOPS and 125 MiB/s, and both can be provisioned independently of size (up to 80,000 IOPS and 2,000 MiB/s), so 9,000 IOPS and 200 MiB/s fit; it gives single-digit-millisecond latency and is the lowest-cost SSD type.
- **b** ❌ st1 sells sequential throughput; small random database I/O performs poorly on HDD media, so it cannot sustain 9,000 random IOPS.
- **c** ❌ io2 would meet the load, but its premium buys sub-millisecond consistency and IOPS beyond gp3's ceiling, neither of which this workload needs.
- **d** ❌ Instance store is ephemeral and loses its data on a stop or hardware failure, breaking the stated persistence requirement.

**The concept.** EBS volume selection is a ladder that starts at gp3: it is the cost-effective default for boot volumes and general databases, and you only climb to io2 when the stated requirement exceeds what gp3 can provision.

**Why this is correct.** The numbers decide it. Random I/O at 9,000 IOPS rules out HDD media, and the persistence requirement rules out instance store. Between the two SSD types, gp3 can be provisioned far past this load (its IOPS and throughput are set independently of volume size) and its single-digit-millisecond latency is explicitly acceptable, so io2's higher price buys nothing the workload asks for. io2 is the tempting overshoot: it works, but MOST cost-effective means the cheapest option that meets the stated need.

**How to reason it out**

1. Classify the access pattern: small random reads and writes from a relational database need SSD, which eliminates st1.
2. Check durability: data must survive stops, which eliminates instance store.
3. Compare the measured load with gp3: 200 MiB/s is above the 125 MiB/s baseline and 9,000 IOPS above the 3,000 baseline, but both are well within what gp3 can be provisioned to.
4. Check the latency requirement: single-digit milliseconds is acceptable, so io2's sub-millisecond consistency is not needed.
5. Apply the cost qualifier: gp3 is the cheapest volume that meets every stated requirement.

> **Exam tip:** Under a cost qualifier, pick gp3 whenever it meets the stated performance — io2 is only justified when the requirement exceeds gp3.

</details>

📖 Learn this topic: [High-Performing Storage: S3 vs EBS vs EFS, Volume Types, and Hybrid Options](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 13

*Design High-Performing Architectures*

**A payments company runs a mission-critical OLTP database on a single EC2 instance. The database sustains about 120,000 random IOPS around the clock, and the business requires storage latency that stays consistently below one millisecond, including at peak. The data must persist independently of the instance. Which storage configuration meets these requirements?** Choose one.

- **a.** An Amazon EFS file system using Elastic throughput mode
- **b.** One io2 Block Express volume provisioned at 120,000 IOPS
- **c.** One gp3 volume with IOPS and throughput set to their maximums
- **d.** Several gp3 volumes striped as RAID 0 to add up enough IOPS

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ EFS is network file storage with roughly millisecond-level reads and slower writes, a poor fit for a latency-critical database data directory.
- **b** ✅ io2 Block Express provisions up to 256,000 IOPS on one volume and is designed for consistent sub-millisecond latency, meeting both requirements durably.
- **c** ❌ A single gp3 volume tops out at 80,000 IOPS and is designed for single-digit-millisecond latency, so it misses both stated requirements.
- **d** ❌ Striping can add up to 120,000 IOPS, but each gp3 volume still delivers single-digit-millisecond latency, so the consistent sub-millisecond requirement is not met.

**The concept.** io2 Block Express is the top rung of the EBS ladder: it is justified when the workload needs sustained IOPS beyond a single gp3 volume's ceiling or consistent sub-millisecond latency, and this scenario needs both.

**Why this is correct.** Two requirements combine. 120,000 IOPS exceeds the 80,000-IOPS maximum of a single gp3 volume, and consistent sub-millisecond latency is outside gp3's single-digit-millisecond design. RAID 0 across gp3 volumes is the clever-sounding workaround: it fixes the IOPS number but not the latency class, and it multiplies failure points. io2 Block Express provisions up to 256,000 IOPS on one volume with sub-millisecond latency, so it meets both. EFS is network file storage and is the wrong shape for database block I/O.

**How to reason it out**

1. Fix the storage type: a single-instance database needs durable block storage, so EBS.
2. Compare the IOPS figure with a single gp3 volume's ceiling (80,000): it is exceeded.
3. Check the latency requirement: consistent sub-millisecond latency is io2 territory, not gp3, even when gp3 volumes are striped.
4. Eliminate EFS as network file storage for a latency-critical block workload.
5. Choose io2 Block Express provisioned to the measured IOPS.

> **Exam tip:** Striping gp3 can add IOPS but not lower latency; sustained IOPS beyond gp3 plus consistent sub-millisecond latency is the io2 Block Express signature.

</details>

📖 Learn this topic: [High-Performing Storage: S3 vs EBS vs EFS, Volume Types, and Hybrid Options](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 14

*Design High-Performing Architectures*

**A web application runs on an Auto Scaling group of EC2 instances behind an Application Load Balancer, and its traffic rises and falls unpredictably through the day. The operations team wants the group to add and remove instances so that fleet CPU stays near 50 percent, and it wants the fewest scaling settings to design and maintain. Which scaling configuration should a solutions architect recommend?** Choose one.

- **a.** Step scaling policies on CloudWatch alarms at 60 and 80 percent CPU, each adding a set number of instances
- **b.** Scheduled actions that set the desired capacity for each hour, sized from last month's CPU data
- **c.** A target tracking policy on the group's average CPU utilization metric with a target value of 50
- **d.** A predictive scaling policy that forecasts CPU utilization and launches instances ahead of the forecast

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Step scaling can approximate a CPU goal, but someone must design and maintain the alarm thresholds and the adjustment for each step — more settings than the requirement allows.
- **b** ❌ Scheduled actions follow the clock, not live load. With unpredictable traffic, hourly sizes drift away from the 50 percent goal and must be re-planned by hand.
- **c** ✅ Target tracking needs one metric and one value; Auto Scaling creates and manages the alarms and adds or removes instances to hold the target in both directions.
- **d** ❌ Predictive scaling forecasts recurring patterns from history. Traffic here is unpredictable, and a forecast alone does not correct for load it did not expect.

**The concept.** Target tracking is the default, AWS-recommended dynamic scaling policy: you choose a metric and a target value, and Auto Scaling manages the alarms and capacity changes needed to keep the metric at that value.

**Why this is correct.** The requirement is a steady utilization goal under unpredictable load with the least configuration. Target tracking is exactly that: one policy, one metric, one number, and the group scales out and in by itself. Step scaling is the workable but heavier option: it can approximate the same behavior only after someone designs the alarm thresholds and per-step adjustments and keeps tuning them. Scheduled actions react to the calendar rather than the load, so they cannot hold a utilization target when traffic deviates. Predictive scaling needs a recurring pattern to forecast; with unpredictable traffic it has nothing reliable to learn, and it does not react to unexpected load on its own.

**How to reason it out**

1. Identify the goal: hold one metric at one value as load moves in both directions.
2. Note the constraints: traffic is unpredictable and the configuration must be minimal.
3. Match a hold-a-value goal to target tracking, which manages its own alarms.
4. Eliminate step scaling (more settings), scheduled actions (clock, not load) and predictive scaling (needs a recurring pattern).

> **Exam tip:** Keep a metric at a value with the least setup is target tracking; step scaling is the higher-maintenance way to get a similar result.

</details>

📖 Learn this topic: [Elastic Compute: Auto Scaling Policies, Lambda Tuning, and Compute Selection](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-elastic-compute-solutions/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 15

*Design High-Performing Architectures*

**A retailer is moving its order-management application to AWS. Each order is written as one transaction that updates inventory, payment and shipping records together. Finance analysts also run unplanned SQL reports against the live data that join those records in new ways every week. Which database should a solutions architect choose?** Choose one.

- **a.** Amazon DynamoDB, using TransactWriteItems for each order and a global secondary index per report
- **b.** Amazon Aurora PostgreSQL, with the inventory, payment and shipping data in normalized tables
- **c.** Amazon Redshift as the system of record, loading each order and running the reports in place
- **d.** Amazon DocumentDB, storing each order as one document with its inventory and shipping data

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ DynamoDB transactions do give all-or-nothing writes across items, but DynamoDB has no joins: every new report shape would need a new index designed in advance or a full-table scan, which fails unplanned join queries.
- **b** ✅ A relational engine handles both halves: ACID transactions across several tables for each order, and ad hoc SQL with multi-table joins for the analysts, on one managed database.
- **c** ❌ Redshift runs analytical joins well, but it is a columnar data warehouse built for bulk loads and large scans, not for a high rate of small order transactions as the operational system of record.
- **d** ❌ Embedding related data in one document suits access patterns known in advance; reports that join inventory, payment and shipping records in new ways every week are the relational engine's strength.

**The concept.** The first database decision on SAA-C03 is the type, read from the access pattern: multi-table transactions together with ad hoc SQL joins point to a relational engine such as Aurora or RDS.

**Why this is correct.** Two requirements decide it. Orders are multi-table transactions, and analysts run reports whose joins are not known in advance. A relational engine like Aurora PostgreSQL provides both: ACID transactions spanning tables, and full SQL so any new join can be written at report time. DynamoDB is the tempting option because it now supports transactions (TransactWriteItems), so the first requirement alone would not rule it out; the second does, because DynamoDB has no join operator and its queries must be designed around keys and indexes ahead of time. Redshift supports the joins but is an analytics warehouse, not the transactional system of record for individual orders. DocumentDB favours embedding related data per document for known access patterns, which works against ad hoc cross-entity reporting.

**How to reason it out**

1. List the access-pattern facts: transactions across several record types, and unplanned join queries.
2. Note that transactions alone no longer exclude DynamoDB, because it supports transactional writes.
3. Apply the second fact: ad hoc joins need a relational query engine, which removes DynamoDB and DocumentDB.
4. Separate OLTP from OLAP: Redshift suits the reports but not the order transactions as the system of record.
5. Choose the managed relational engine, Aurora PostgreSQL.

> **Exam tip:** Ad hoc SQL joins plus multi-table transactions mean a relational engine; DynamoDB transactions do not give you joins.

</details>

📖 Learn this topic: [High-Performing Databases: Aurora vs RDS, DynamoDB, DAX and ElastiCache](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 16

*Design High-Performing Architectures*

**A news publisher serves images, video clips, and JavaScript bundles from an Application Load Balancer in us-east-1 to readers worldwide. Page loads are slow for readers in Asia and Europe, and the origin fleet is strained because it answers the same asset requests over and over. Which change reduces latency for distant readers and also reduces the request load on the origin?** Choose one.

- **a.** Place an AWS Global Accelerator accelerator in front of the existing Application Load Balancer
- **b.** Add Amazon Route 53 latency-based routing records for the existing Application Load Balancer
- **c.** Replace the existing Application Load Balancer with a Network Load Balancer using cross-zone balancing
- **d.** Place an Amazon CloudFront distribution in front of the existing Application Load Balancer

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Global Accelerator brings readers onto the AWS backbone at a nearby edge, but it caches nothing, so every asset request still reaches the origin fleet.
- **b** ❌ Latency-based routing chooses between deployments in several Regions; with one origin Region there is nothing to choose between, and no request is cached.
- **c** ❌ A Network Load Balancer spreads connections across targets inside one Region; it does not shorten the distance to readers or reduce how many requests reach the fleet.
- **d** ✅ CloudFront caches the static assets at edge locations, so repeat requests are answered near the reader and never reach the origin, fixing both latency and origin load.

**The concept.** Amazon CloudFront is a content delivery network: it caches content at edge locations, which both serves users from a nearby edge and removes repeated requests from the origin.

**Why this is correct.** The stem gives two symptoms: distance-driven latency for global readers and an origin answering identical requests repeatedly. Only edge caching addresses both: an asset is fetched from the origin once per edge location and then served locally to every later reader. Global Accelerator is the tempting near-twin because it also improves global performance from the edge, but it forwards every connection to the endpoint and caches nothing, so origin load is unchanged. Latency-based routing needs several Regional deployments to choose from, and a different load balancer type changes nothing about distance or request volume.

**How to reason it out**

1. Classify the content: static, cacheable HTTP assets requested repeatedly.
2. Note the two symptoms: latency for distant readers and origin strain from repeated requests.
3. Pick the mechanism that addresses both: edge caching in CloudFront.
4. Reject Global Accelerator because it accelerates connections but never caches.
5. Reject DNS routing and load-balancer changes because they leave request volume at the origin unchanged.

> **Exam tip:** Cacheable HTTP content plus "reduce load on the origin" is CloudFront; Global Accelerator improves the path but forwards every request.

</details>

📖 Learn this topic: [Scalable Networks: CloudFront vs Global Accelerator, Transit Gateway, ALB vs NLB](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-network-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 17

*Design Cost-Optimized Architectures*

**A company generates monthly financial reports in Amazon S3. Each report is downloaded several times during its first 30 days, then accessed a few times per month for the next two years and must load with millisecond latency whenever it is requested. Which storage design meets these requirements MOST cost-effectively?** Choose one.

- **a.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Glacier Flexible Retrieval 30 days after creation.
- **b.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Standard-IA 30 days after creation.
- **c.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Glacier Instant Retrieval 30 days after creation.
- **d.** Keep new reports in S3 Standard and leave them there, with no lifecycle rule, until the two-year retention ends.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Glacier Flexible Retrieval restores asynchronously in minutes to hours, which violates the requirement that reports load with millisecond latency.
- **b** ✅ Standard absorbs the frequent first-month access without retrieval fees, and Standard-IA then provides millisecond access at a much lower storage price for monthly reads, cleanly satisfying its 30-day minimum duration.
- **c** ❌ Glacier Instant Retrieval is priced for data touched about once a quarter; at a few reads per month its higher retrieval fees erode the storage savings compared with Standard-IA.
- **d** ❌ This meets every access requirement but pays the highest per-GB storage price for two years when Standard-IA would serve the same monthly, millisecond reads for much less after day 30.

**The concept.** S3 class selection maps each phase of an access pattern to the cheapest class that still meets it. Standard-IA suits data accessed roughly monthly that must stay immediately available, with a 30-day minimum storage duration and a per-GB retrieval fee.

**Why this is correct.** The reports have two phases: frequent access for 30 days, then occasional access needing millisecond reads. Standard is right for phase one because IA-class retrieval fees would dominate under frequent access. Transitioning to Standard-IA at day 30 is the earliest supported IA transition, satisfies the 30-day minimum duration, and keeps millisecond access at a much lower storage price. Glacier Instant Retrieval keeps millisecond access but its retrieval fees are priced for roughly quarterly access, so a-few-times-per-month reads make it more expensive than Standard-IA. Glacier Flexible Retrieval fails the millisecond requirement outright because restores are asynchronous. Staying in Standard for two years over-delivers: it meets every requirement but is not the cheapest option that does.

**How to reason it out**

1. Split the lifecycle into phases: frequent access for 30 days, then a-few-times-per-month access with millisecond reads for two years.
2. Assign Standard to the frequent phase, since any IA or Glacier class would charge retrieval fees on every download.
3. For the second phase, require millisecond access, which eliminates Glacier Flexible Retrieval.
4. Compare Standard-IA and Glacier Instant Retrieval at the stated frequency: monthly access favors Standard-IA because Glacier Instant Retrieval retrieval fees are tuned for quarterly access.
5. Verify the trap: the 30-day transition satisfies Standard-IA's 30-day minimum duration, so no minimum-duration penalty applies.

> **Exam tip:** Monthly access with millisecond reads is the Standard-IA pattern; quarterly access is Glacier Instant Retrieval's.

</details>

📖 Learn this topic: [Cost-Optimized Storage: S3 Classes, Lifecycle Policies, and EBS Economics](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 18

*Design Cost-Optimized Architectures*

**A data processing job writes intermediate artifacts to Amazon S3. The artifacts are rarely read, but when the job retries a failed step it must read the relevant artifact immediately. A cleanup requirement deletes every artifact 10 days after creation. Which storage configuration is MOST cost-effective?** Choose one.

- **a.** Store the artifacts in S3 Glacier Flexible Retrieval with a lifecycle expiration rule that deletes them after 10 days.
- **b.** Store the artifacts in S3 Standard-IA with a lifecycle expiration rule that deletes them after 10 days.
- **c.** Store the artifacts in S3 One Zone-IA with a lifecycle expiration rule that deletes them after 10 days.
- **d.** Store the artifacts in S3 Standard with a lifecycle expiration rule that deletes them after 10 days.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Glacier Flexible Retrieval bills a 90-day minimum and its objects must be restored before they can be read, which fails the immediate-read requirement on a retry.
- **b** ❌ Standard-IA bills a 30-day minimum storage duration, so every artifact deleted at day 10 is charged for 30 days at the IA rate, which costs more than 10 days in Standard.
- **c** ❌ One Zone-IA has the same 30-day minimum as Standard-IA; 30 billed days at its rate still exceed 10 days in Standard, so the lower per-GB price never pays off.
- **d** ✅ Standard has no minimum storage duration and no retrieval fee, so the artifacts are billed for exactly the 10 days they exist and can be read instantly on a retry.

**The concept.** Every class below S3 Standard bills a minimum storage duration (30 days for the IA classes, 90 for Glacier Instant and Flexible Retrieval, 180 for Deep Archive). Data deleted before that minimum is charged for the full minimum anyway.

**Why this is correct.** The artifacts live for 10 days. In Standard they are billed for those 10 days only. Both IA classes charge their 30-day minimum, so 30 days at roughly half the Standard rate costs more than 10 days at the full rate, and reads add a retrieval fee on top. Glacier Flexible Retrieval also needs a restore before any read, which breaks the immediate-read requirement for retries. "Rarely read" tempts candidates toward an infrequent-access class, but for short-lived data the minimum duration decides, not the access frequency.

**How to reason it out**

1. Note the lifetime first: every artifact is deleted at day 10.
2. Compare that lifetime with each class minimum: 30 days for both IA classes, 90 days for Glacier Flexible Retrieval.
3. Price it: 30 billed days at the IA rate exceed 10 days at the Standard rate.
4. Check the access requirement: an immediate read on retry rules out any class that needs a restore.
5. Choose Standard with an expiration rule.

> **Exam tip:** Data deleted before a class minimum duration belongs in S3 Standard, however rarely it is read.

</details>

📖 Learn this topic: [Cost-Optimized Storage: S3 Classes, Lifecycle Policies, and EBS Economics](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 19

*Design Cost-Optimized Architectures*

**An application runs on EC2 with a steady floor of 10 instances 24/7 year-round, up to 15 additional instances during business hours, and a nightly batch job on a separate fleet that reprocesses safely from checkpoints in S3. Everything currently runs On-Demand. Which purchasing design is MOST cost-effective?** Choose one.

- **a.** Run all three layers on Spot Instances to maximize the discount.
- **b.** Keep the entire workload On-Demand for maximum flexibility.
- **c.** Buy Standard Reserved Instances covering all 25 web-tier instances.
- **d.** Buy a Savings Plan sized to the 10-instance baseline, run the business-hours layer On-Demand behind Auto Scaling, and move the batch fleet to Spot Instances.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ The always-on baseline and business-hours tier cannot tolerate a two-minute reclaim, so Spot under-delivers on availability no matter how cheap it is.
- **b** ❌ The 10-instance floor has run all year, so paying full price for guaranteed usage forfeits up to 72 percent in commitment savings for no benefit.
- **c** ❌ Committing to the peak pays reserved rates for 15 instances that idle outside business hours; commitments only pay off on steady utilization.
- **d** ✅ Each layer gets the deepest discount its reliability allows: up to 72 percent on the guaranteed baseline, full rate only for hours the variable layer runs, and up to 90 percent on the checkpointed batch work.

**The concept.** The canonical compute purchasing answer is layered: commitments (Savings Plans or RIs) sized to the steady baseline, On-Demand behind Auto Scaling for the variable layer, and Spot for interruption-tolerant work.

**Why this is correct.** Decomposing by layer gives each slice the deepest discount its requirements allow. The 10-instance floor is guaranteed usage, so a Savings Plan captures up to 72 percent on spend that would occur anyway. The business-hours layer is variable, so On-Demand behind Auto Scaling pays full rate only for the hours the capacity exists; committing to it would bill reserved capacity idling every night. The batch fleet checkpoints to S3, passing Spot's admission test, so it takes up to 90 percent off. Reserving all 25 instances commits to peak, all On-Demand ignores the baseline's guaranteed savings, and all-Spot puts un-interruptible tiers on reclaimable capacity, which is a requirement violation, not a saving.

**How to reason it out**

1. Decompose the workload into baseline, variable, and interruption-tolerant layers.
2. Cover the measured steady floor with a Savings Plan or Reserved Instances.
3. Run the predictable-but-variable layer On-Demand behind Auto Scaling so it scales away when idle.
4. Move the checkpointed batch fleet to Spot for up to 90 percent off.
5. Reject any option that commits to peak or puts un-interruptible work on Spot.

> **Exam tip:** Commit to the floor, scale the middle On-Demand, and push interruptible work to Spot — never commit to peak.

</details>

📖 Learn this topic: [Cost-Optimized Compute: Spot vs Reserved vs Savings Plans, and Lambda vs EC2](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-compute/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 20

*Design Cost-Optimized Architectures*

**A company runs an internal reporting application on an Amazon RDS for MySQL memory-optimized instance that runs 24/7 at on-demand rates. Analysts use the application only on weekdays from 9 AM to 6 PM, and the database is idle overnight and at weekends. The application must keep MySQL compatibility, and a delay of up to a minute on the first connection after an idle period is acceptable. Which change will reduce the database cost the most?** Choose one.

- **a.** Migrate the database to a provisioned Aurora MySQL cluster and add Aurora Auto Scaling for the Aurora Replicas.
- **b.** Migrate the database to Aurora Serverless v2 for MySQL and set the cluster's minimum capacity to 0 ACUs.
- **c.** Purchase a 1-year Reserved Instance for the current RDS for MySQL instance class and keep the instance running.
- **d.** Move the database to a smaller burstable RDS for MySQL instance class and purchase a Reserved Instance for it.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Aurora Auto Scaling adds and removes read replicas, but the provisioned writer instance still runs and bills around the clock, idle or not.
- **b** ✅ With a 0 ACU minimum, Aurora Serverless v2 pauses automatically when there are no connections and charges no compute while paused; it resumes on the next connection after a resume delay (typically about 15 seconds, longer after more than 24 hours paused, as every weekend here), which the one-minute allowance covers, and it stays MySQL-compatible.
- **c** ❌ A reservation lowers the hourly rate but still pays for every idle night and weekend hour, which is roughly three-quarters of the week here.
- **d** ❌ A smaller class lowers the rate, but a provisioned instance still bills every hour it runs, so the idle majority of the week keeps costing money.

**The concept.** A provisioned relational database bills instance-hours whether or not queries arrive. When a workload is idle most of the week, the cheapest design is one that stops charging for compute while idle, not one that discounts the idle hours.

**Why this is correct.** The workload is busy about 45 hours of a 168-hour week, so most of its bill is idle time. Aurora Serverless v2 bills per second of consumed capacity, and with a minimum capacity of 0 ACUs it auto-pauses after a period with no connections, leaving only storage charges until the next connection resumes it. Resuming typically takes about 15 seconds, and 30 seconds or more after a pause longer than 24 hours (every weekend here); the stem accepts up to a minute and requires MySQL compatibility, which Aurora MySQL keeps. The Reserved Instance and the smaller burstable class both reduce the hourly rate but still pay every idle hour. Aurora Auto Scaling only scales replicas; the writer still runs continuously.

**How to reason it out**

1. Work out the workload shape: about a quarter of the week busy, the rest idle.
2. Note the constraints: MySQL compatibility, and a delay of up to a minute after idle periods is acceptable.
3. Eliminate options that keep a provisioned writer or instance billing through idle hours, however discounted.
4. Choose Aurora Serverless v2 with a 0 ACU minimum so that compute stops billing while the database is paused.

> **Exam tip:** For an idle-heavy relational workload that can tolerate a brief resume delay, Aurora Serverless v2 scaled to 0 ACUs beats any discount on a provisioned instance.

</details>

📖 Learn this topic: [Cost-Optimized Databases: DynamoDB vs RDS, Aurora Serverless, and Caching](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

[← Back to the SAA-C03 study guide](README.md)
