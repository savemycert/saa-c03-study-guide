# SAA-C03 Sample Questions with Answers

5 worked practice questions for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [SAA-C03 practice questions](https://www.savemycert.com/practice/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide) on SaveMyCert.

## Question 1

*Design Secure Architectures*

**A company manages 40 AWS accounts in AWS Organizations, grouped into a Development OU and a Production OU. The security team requires that no user in any Development account, including account administrators who hold full IAM permissions, can stop or delete AWS CloudTrail trails. What is the MOST effective way to enforce this requirement?** Choose one.

- **a.** Attach an IAM policy that denies cloudtrail:StopLogging and cloudtrail:DeleteTrail to every IAM user and role in each Development account.
- **b.** Attach a service control policy to the Development OU that denies cloudtrail:StopLogging and cloudtrail:DeleteTrail.
- **c.** Attach a permissions boundary that denies CloudTrail modifications to all IAM roles in the Development accounts.
- **d.** Deploy an AWS Config rule in each Development account that detects when a CloudTrail trail is stopped and notifies the security team.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Account administrators hold iam:* permissions, so they can detach or edit any IAM policy deployed inside their own account, defeating the control.
- **b** ✅ An SCP is enforced from the organization, above the member accounts, so it binds every principal in every account under the OU, including administrators and member-account root users, and no one inside those accounts can remove it.
- **c** ❌ Permissions boundaries apply per identity and are managed inside each account, so administrators who control IAM could remove or replace the boundary.
- **d** ❌ A Config rule is a detective control; it reports the violation after CloudTrail has already been stopped rather than preventing the action.

**The concept.** Restrictions that must bind account administrators cannot live inside the accounts those administrators control. Service control policies are applied from AWS Organizations, above the member accounts, and set the maximum permissions available to every principal within them.

**Why this is correct.** The SCP on the Development OU wins because it is enforced at a layer no Development-account principal can reach: it inherits to every account in the OU, constrains even member-account root users, and denies the CloudTrail actions at the API regardless of what IAM policies allow. The per-account IAM deny policy fails because administrators with iam:* can simply detach it. The permissions boundary fails for the same reason at a different layer: boundaries are per-identity objects managed inside the account, so an admin can strip them off. The Config rule is detective, not preventive; it tells you logging was disabled instead of stopping it, which does not satisfy a requirement that no one can perform the action.

**How to reason it out**

1. Identify the requirement type: a restriction that must hold against administrators, not a grant of access.
2. Eliminate any control managed inside the member accounts (IAM policies, permissions boundaries), because admins with iam:* can modify or remove it.
3. Eliminate detective options (Config rules) because the requirement is prevention, not notification.
4. Choose an SCP attached at the OU level so it inherits to all Development accounts and binds every principal, including root.
5. Confirm the SCP uses an explicit deny for the specific CloudTrail actions, which overrides any allow anywhere.

> **Exam tip:** When a restriction must bind administrators or root in member accounts, the answer is an SCP applied from the organization, never an IAM policy inside the account.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 2

*Design Resilient Architectures*

**A photo-sharing startup runs a web tier that synchronously calls a fleet of EC2 instances to generate image renditions after each upload. When a post goes viral, the rendition fleet saturates, upload requests time out, and some uploads are never processed. The company needs a redesign that guarantees no upload job is lost and handles unpredictable spikes MOST cost-effectively. What should a solutions architect recommend?** Choose one.

- **a.** Place an Amazon SQS queue between the web tier and the rendition fleet, and run the fleet in an Auto Scaling group that scales on the queue backlog.
- **b.** Provision the rendition fleet with enough EC2 instances to handle the largest expected spike at all times.
- **c.** Publish each upload job to an Amazon SNS topic and subscribe the rendition instances as HTTP endpoints.
- **d.** Put an Application Load Balancer in front of the rendition fleet and increase the web tier request timeout.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The queue durably stores every job until a worker deletes it after success, so nothing is lost, and workers scale out on backlog during a spike and back in afterward, so the company pays only for capacity it uses.
- **b** ❌ Static peak provisioning keeps the synchronous coupling, still drops work if a spike exceeds the estimate, and pays for idle capacity around the clock, failing the cost qualifier.
- **c** ❌ SNS pushes at its own pace instead of letting saturated workers pull at theirs, and once delivery retries to an overwhelmed endpoint are exhausted the message is gone; it is not a durable consumer-paced buffer.
- **d** ❌ A load balancer spreads synchronous requests but stores nothing; when every instance is saturated, requests still time out and the jobs are still lost.

**The concept.** Inserting a queue between tiers converts a traffic spike into a durable backlog that a worker tier drains at its own pace. This decouples the tiers so each scales on its own signal and no work is lost while consumers catch up.

**Why this is correct.** The SQS-plus-Auto-Scaling design wins on both stated requirements: SQS persists every job until a worker successfully processes and deletes it (no lost uploads), and a scaling policy driven by queue depth adds workers only while a backlog exists (cost-effective for unpredictable spikes). Peak provisioning is the classic anti-pattern the cost qualifier punishes: it keeps the fragile synchronous call path, has a hard ceiling, and bills for idle capacity every quiet hour. SNS looks similar but is push-based delivery without consumer pacing; an overwhelmed HTTP subscriber eventually exhausts retries and drops messages, so it cannot guarantee zero loss. The load balancer option redistributes the same synchronous load without adding any storage, so a fleet-wide saturation still times out and loses the job.

**How to reason it out**

1. Diagnose the failure mode: synchronous calls into a saturated processing tier drop work during spikes, which is tight coupling.
2. Introduce a durable intermediary: an SQS queue holds each job until a worker finishes and deletes it, satisfying the no-loss requirement.
3. Scale the consumer on the right signal: a policy on queue backlog grows the worker Auto Scaling group during spikes and shrinks it after.
4. Apply the qualifier: pay-per-use elastic workers beat always-on peak capacity, so the queue-based design is the MOST cost-effective option that also works.

> **Exam tip:** When spikes cause lost work between tiers, buffer with SQS and scale workers on queue depth instead of resizing the coupled tiers.

</details>

📖 Learn this topic: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 3

*Design High-Performing Architectures*

**A company is deploying a new relational database on a single EC2 instance. Monitoring of the current environment shows a moderate, steady level of random IOPS that is well within the range a general purpose SSD volume can deliver. Which EBS volume type meets the performance requirement MOST cost-effectively?** Choose one.

- **a.** General Purpose SSD (gp3)
- **b.** Provisioned IOPS SSD (io2)
- **c.** Throughput Optimized HDD (st1)
- **d.** Instance store on a storage-optimized instance

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ gp3 delivers low-latency SSD performance for random database I/O at balanced cost, and the stated requirement sits comfortably within what it can provision.
- **b** ❌ io2 would certainly meet the requirement, but it is the most expensive volume type and the workload does not need sustained extreme IOPS or guaranteed sub-millisecond consistency.
- **c** ❌ st1 is built for large sequential reads and writes; it performs poorly on the small random I/O pattern of a transactional database.
- **d** ❌ Instance store is ephemeral, so the database files would be lost on an instance stop or hardware failure — unacceptable for a system of record.

**The concept.** EBS volume selection is a ladder that starts at gp3: it is the cost-effective default for boot volumes and general databases, and you only climb to io2 when the stated requirement exceeds what gp3 can provision.

**Why this is correct.** The scenario explicitly places the IOPS demand within general purpose SSD range, so gp3 meets the requirement at the lowest cost among options that actually fit. io2 also works technically but is the classic overpriced distractor under a MOST cost-effective qualifier — you pay provisioned-IOPS prices for headroom the workload never uses. st1 fails on access pattern: HDD volumes sell sequential throughput and are poor at the random I/O a relational database generates. Instance store is the fastest storage available but is ephemeral, so it is disqualified for durable database files regardless of speed.

**How to reason it out**

1. Classify the access pattern: a relational database generates small random reads and writes, which points to SSD, eliminating st1.
2. Check durability: database files must survive instance stops, eliminating ephemeral instance store.
3. Compare the stated requirement to the gp3 envelope: the scenario says the IOPS level fits within general purpose range.
4. Apply the qualifier: with gp3 sufficient, io2 is an overshoot on cost, so gp3 wins.

> **Exam tip:** Under a cost qualifier, pick gp3 whenever it meets the stated performance — io2 is only justified when the requirement exceeds gp3.

</details>

📖 Learn this topic: [High-Performing Storage: S3 vs EBS vs EFS, Volume Types, and Hybrid Options](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 4

*Design Cost-Optimized Architectures*

**A company generates monthly financial reports in Amazon S3. Each report is downloaded several times during its first 30 days, then accessed a few times per month for the next two years and must load with millisecond latency whenever it is requested. Which storage design meets these requirements MOST cost-effectively?** Choose one.

- **a.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Standard-IA 30 days after creation.
- **b.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Glacier Instant Retrieval 30 days after creation.
- **c.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Glacier Flexible Retrieval 30 days after creation.
- **d.** Store all reports in S3 Standard for the full two-year retention period.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Standard absorbs the frequent first-month access without retrieval fees, and Standard-IA then provides millisecond access at a much lower storage price for monthly reads, cleanly satisfying its 30-day minimum duration.
- **b** ❌ Glacier Instant Retrieval is priced for data touched about once a quarter; at a few reads per month its higher retrieval fees erode the storage savings compared with Standard-IA.
- **c** ❌ Glacier Flexible Retrieval restores asynchronously in minutes to hours, which violates the requirement that reports load with millisecond latency.
- **d** ❌ This meets every access requirement but pays the highest per-GB storage price for two years when a cheaper sufficient class exists after day 30.

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

## Question 5

*Design Secure Architectures*

**A solutions architect creates a new member account in AWS Organizations and attaches a service control policy to it that explicitly allows s3:* and ec2:*. A newly created IAM user in the account has no IAM policies attached. When the user attempts to list S3 buckets, the request fails with an access denied error. What explains this behavior?** Choose one.

- **a.** SCPs set the maximum available permissions but never grant access, so the user still needs an identity-based policy that allows the S3 actions.
- **b.** Service control policies take up to 24 hours to propagate to newly created member accounts.
- **c.** The SCP must be attached to the organizational unit rather than directly to the account for it to take effect.
- **d.** Each S3 bucket requires a bucket policy that allows the user before any IAM identity in the account can list buckets.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ An SCP acts as a filter on what IAM policies may allow; with no identity-based allow, the request falls back to the implicit deny even though the SCP permits S3.
- **b** ❌ SCP changes take effect quickly and no such propagation delay exists; more importantly, propagation would not fix the missing identity-based allow.
- **c** ❌ SCPs can attach to the organization root, an OU, or an individual account; direct account attachment is valid, and attachment location does not change the fact that SCPs cannot grant.
- **d** ❌ Within one account, an allow in either the identity policy or a resource policy is sufficient; bucket policies are not a prerequisite when an identity-based policy allows the action.

**The concept.** An SCP never grants permissions. It defines the maximum permissions available to principals in the accounts it applies to, and effective access is the intersection of the SCP and the principal's IAM policies.

**Why this is correct.** The user has no identity-based policy, so nothing has ever allowed the S3 actions; the request is implicitly denied regardless of how generous the SCP is. The SCP full of allows only means that if an IAM policy someday grants S3 actions, the SCP will not block them. The propagation-delay option is a fabricated mechanic and does not address the missing allow. The OU-attachment option is wrong because SCPs attach validly at the root, OU, or account level. The bucket-policy option inverts the within-account union rule: identity-based and resource-based policies each independently suffice to allow, so no bucket policy is required when an identity policy grants access.

**How to reason it out**

1. Start from IAM's default: every request is implicitly denied until some policy explicitly allows it.
2. Check for an identity-based or resource-based allow for the user; here there is none.
3. Recognize that the SCP is a ceiling, not a grant: it filters allows but cannot create them.
4. Conclude the fix is attaching an identity-based policy allowing the needed S3 actions, which the SCP will then permit.

> **Exam tip:** SCPs cap permissions; they never grant them — a principal with no IAM allow gets nothing no matter what the SCP permits.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

[← Back to the SAA-C03 study guide](README.md)
