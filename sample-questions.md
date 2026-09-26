# SAA-C03 Sample Questions with Answers

20 worked practice questions for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam. Try each one before you open the answer.

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

*Design Secure Architectures*

**A solutions architect creates a new member account in AWS Organizations and attaches a service control policy to it that explicitly allows s3:* and ec2:*. A newly created IAM user in the account has no IAM policies attached. When the user attempts to list S3 buckets, the request fails with an access denied error. What explains this behavior?** Choose one.

- **a.** Service control policies take up to 24 hours to propagate to newly created member accounts.
- **b.** The SCP must be attached to the organizational unit rather than directly to the account for it to take effect.
- **c.** Each S3 bucket requires a bucket policy that allows the user before any IAM identity in the account can list buckets.
- **d.** SCPs set the maximum available permissions but never grant access, so the user still needs an identity-based policy that allows the S3 actions.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ SCP changes take effect quickly and no such propagation delay exists; more importantly, propagation would not fix the missing identity-based allow.
- **b** ❌ SCPs can attach to the organization root, an OU, or an individual account; direct account attachment is valid, and attachment location does not change the fact that SCPs cannot grant.
- **c** ❌ Within one account, an allow in either the identity policy or a resource policy is sufficient; bucket policies are not a prerequisite when an identity-based policy allows the action.
- **d** ✅ An SCP acts as a filter on what IAM policies may allow; with no identity-based allow, the request falls back to the implicit deny even though the SCP permits S3.

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

## Question 3

*Design Secure Architectures*

**A security team attaches a service control policy to the root of its organization that denies all actions in AWS Regions other than eu-west-1 and eu-central-1. One month later, a review finds new EC2 instances running in us-east-1 that were launched by administrators after the SCP took effect. All of the instances are in a single account. What is the MOST likely explanation?** Choose one.

- **a.** Amazon EC2 is a global service that is exempt from Region-based SCP conditions.
- **b.** The administrators used the account's root user, and SCPs do not apply to root users.
- **c.** SCPs attached to the organization root apply only to OUs, not to accounts placed directly under the root.
- **d.** The instances were launched in the organization's management account, which SCPs do not affect.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ EC2 is a regional service; Region-restriction SCPs commonly exempt a handful of truly global services such as IAM, but EC2 is not one of them.
- **b** ❌ SCPs do constrain the root user of member accounts; root exemption applies only in the management account.
- **c** ❌ SCPs attached to the organization root inherit to every OU and every account beneath it; there is no gap for accounts directly under the root.
- **d** ✅ SCPs have no effect on the management account — not on its users, roles, or root user — so it is the one account a root-level Region-deny SCP cannot constrain.

**The concept.** SCPs apply to every member account in the organization, including member-account root users, but they have no effect whatsoever on the management account — not on its users, roles, or root user.

**Why this is correct.** Because the SCP sits at the organization root, every member account and OU inherits it, and it binds all principals in those accounts including root. The single account that can still act outside the allowed Regions is the management account, which no SCP can constrain — which is also why AWS recommends running no workloads there. The member-root option inverts the actual rule: member-account root users are constrained by SCPs. The root-attachment option is wrong because root-level SCPs inherit down to everything. The EC2-exemption option is a distractor built on the real practice of exempting global services like IAM in Region-deny SCPs; EC2 is regional and gets no such exemption.

**How to reason it out**

1. Confirm the SCP scope: attached at the organization root, it inherits to all OUs and member accounts.
2. Recall that within member accounts, SCPs bind every principal, including administrators and root.
3. Identify the one principal set SCPs can never touch: everything inside the management account.
4. Match the evidence — one unaffected account launching resources post-SCP — to the management account.
5. Note the operational lesson: keep workloads and daily activity out of the management account.

> **Exam tip:** SCPs bind member accounts and their root users but never the management account — keep workloads out of it.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 4

*Design Secure Architectures*

**A company runs a two-tier web application in a single VPC. The web servers and an RDS MySQL database both run in a public subnet, and the database's security group allows inbound traffic on port 3306 from 0.0.0.0/0 so that the application can always reach it. Security now requires that the database be unreachable from the internet. Which redesign is MOST secure?** Choose one.

- **a.** Move the database to a subnet with no internet gateway or NAT route and allow inbound port 3306 only from the web tier's security group, referenced by ID.
- **b.** Keep the current network layout and require all database connections to use SSL/TLS.
- **c.** Keep the database in the public subnet but change its security group to allow port 3306 only from the web tier's public IP addresses.
- **d.** Keep the database in the public subnet and add a network ACL that denies port 3306 from all sources except the web servers' IP addresses.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The isolated subnet removes any internet path at the routing layer, and the security-group reference restricts access to the web tier by relationship rather than by IP.
- **b** ❌ Encryption in transit protects data on the wire but does nothing about internet reachability — the database can still be probed and attacked from anywhere.
- **c** ❌ The database remains in an internet-routable subnet, the rule breaks whenever web-server IPs change, and one security group misedit re-exposes it to the internet.
- **d** ❌ A NACL bolted onto a public subnet leaves the exposed placement in place and depends on maintaining an IP list that scaling events invalidate.

**The concept.** A subnet's route table defines its exposure: no internet gateway route and no NAT route means nothing in the subnet can reach or be reached from the internet. Combined with a security group that references the calling tier's group by ID, this removes exposure at the design level rather than patching it.

**Why this is correct.** Moving the database into an isolated subnet eliminates internet reachability structurally — there is simply no route — and the SG-by-ID rule means any instance carrying the web tier's security group can connect while nothing else can, surviving Auto Scaling with zero rule maintenance. The tightened-CIDR option reduces the exposure without eliminating it: placement is still public and reachability still hinges on one rule staying correct. The NACL option layers a stateless, IP-based filter over the same flawed placement. The SSL option answers a different question entirely — confidentiality, not reachability — and is the classic partial-fix distractor.

**How to reason it out**

1. Identify both flaws: internet-routable subnet placement and a 0.0.0.0/0 database rule.
2. Fix placement first: put the database in a subnet whose route table has no IGW or NAT entry.
3. Fix access next: allow the database port only from the web tier's security group, referenced by ID.
4. Prefer structural removal of exposure over options that merely narrow it (CIDR rules, NACLs).
5. Discard options that address confidentiality (TLS) when the requirement is reachability.

> **Exam tip:** Databases belong in isolated subnets with inbound allowed only from the application tier's security group by ID — never in public subnets with tightened IP rules.

</details>

📖 Learn this topic: [Securing Workloads: VPC Design, Security Groups vs NACLs, WAF and Shield](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-vpc-security-secure-workloads/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 5

*Design Secure Architectures*

**A financial services company stores compliance reports in Amazon S3 with server-side encryption. The security team must author the policy that controls exactly which IAM principals can use the encryption key, and encrypted snapshots of the reporting servers must be shared with an external auditor's AWS account. Which type of encryption key meets these requirements?** Choose one.

- **a.** AWS managed KMS keys such as aws/s3 and aws/ebs
- **b.** SSE-S3 keys fully managed by Amazon S3
- **c.** A customer managed KMS key
- **d.** AWS owned keys managed by each service

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ AWS managed keys are visible in your account, but you cannot edit their key policies and they cannot be used in cross-account scenarios, so both requirements fail.
- **b** ❌ SSE-S3 keys have no key policy at all and provide no cross-account key sharing, so the security team has no control point.
- **c** ✅ Only customer managed keys let you author the key policy and grant another AWS account permission to use the key, which is required for both the access-control and snapshot-sharing requirements.
- **d** ❌ AWS owned keys belong to the service itself; you never see them, cannot attach a policy to them, and cannot use them across accounts.

**The concept.** KMS offers three ownership levels: AWS owned keys (invisible, no control), AWS managed keys (visible but with uneditable policies and no cross-account use), and customer managed keys (you author the key policy, control grants, and can share across accounts).

**Why this is correct.** The scenario states two requirements that only a customer managed key satisfies: writing the key policy that restricts which principals may use the key, and cross-account access so the auditor's account can decrypt shared snapshots. AWS managed keys encrypt data just as well, which is why they appear as a distractor, but their key policies cannot be edited and they cannot be used by another account. AWS owned keys and SSE-S3 keys offer even less control, with no visible key policy at all.

**How to reason it out**

1. Scan the stem for control words: 'author the policy' and 'shared with an external account' are both key-ownership requirements.
2. Eliminate AWS owned keys and SSE-S3 immediately because neither exposes a key policy you can write.
3. Eliminate AWS managed keys because their policies are fixed by AWS and they never work cross-account.
4. Select the customer managed key, the only rung of the key-ownership ladder that supports custom key policies and cross-account grants.

> **Exam tip:** Any requirement to edit key policies, control rotation, disable a key, or share encrypted data across accounts points to a customer managed KMS key.

</details>

📖 Learn this topic: [AWS Data Security Controls: KMS Encryption, S3 Encryption, and Key Management](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-data-security-encryption-kms/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 6

*Design Secure Architectures*

**A financial services company keeps compliance logs in an S3 bucket in a dedicated audit account within its AWS organization. Regulators require a guarantee that no principal in any member account — including member-account root users — can delete the bucket, and the company wants this guarantee to also cover new workloads it deploys in the future. Which combination of actions should a solutions architect take? (Select TWO.)** Choose 2.

- **a.** Attach a service control policy with an explicit deny for s3:DeleteBucket on the audit bucket to the organizational units that contain the member accounts.
- **b.** Run all current and future workloads in member accounts rather than in the organization's management account.
- **c.** Attach an IAM policy with an explicit deny for s3:DeleteBucket to the root user of each member account.
- **d.** Apply a permissions boundary that denies s3:DeleteBucket to the root user of each member account.
- **e.** Attach the same service control policy to the management account so that its principals are equally restricted.

<details>
<summary>Show answer and explanation</summary>

**Answer: a, b**

- **a** ✅ An SCP explicit deny inherits to every account in the OUs and binds all principals there, including administrators and member-account root users, and cannot be removed from inside those accounts.
- **b** ✅ SCPs have no effect on the management account, so any workload or principal there would sit outside the guardrail; keeping workloads in member accounts keeps them all inside SCP enforcement.
- **c** ❌ Root users are not governed by IAM identity-based policies; you cannot attach IAM policies to root, so this control is impossible to implement.
- **d** ❌ Permissions boundaries attach only to IAM users and roles; they cannot be applied to root users at all.
- **e** ❌ SCPs have no effect on the management account regardless of where they are attached, so this adds no protection.

**The concept.** Only SCPs can constrain member-account root users, and no control of any kind constrains the management account. A guardrail design must therefore combine an SCP with the discipline of keeping workloads out of the management account.

**Why this is correct.** The SCP explicit deny on the member-account OUs is the only mechanism that reaches every principal in those accounts, root included, and it is enforced from above the accounts so no insider can lift it. Keeping workloads in member accounts is the necessary second half: the management account is immune to SCPs, so any future workload placed there would escape the guarantee — the requirement explicitly covers future workloads. The IAM-policy-on-root and boundary-on-root options fail on the same precise fact: root users are not IAM identities and accept neither identity policies nor permissions boundaries. Attaching the SCP to the management account is a no-op because SCPs simply never evaluate against management-account principals.

**How to reason it out**

1. Translate the requirement: a preventive restriction binding all member-account principals, including root, now and for future workloads.
2. Select the only layer that constrains member root users: an SCP with an explicit deny, attached to the relevant OUs.
3. Recall the SCP blind spot: the management account is never affected, so workloads must not live there.
4. Eliminate options that attach IAM policies or boundaries to root, since root accepts neither.
5. Eliminate attaching the SCP to the management account, which has no effect by design.

> **Exam tip:** Guardrails that must bind root are SCPs on member accounts — and because SCPs never reach the management account, workloads must stay out of it.

</details>

📖 Learn this topic: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 7

*Design Resilient Architectures*

**A photo-sharing startup runs a web tier that synchronously calls a fleet of EC2 instances to generate image renditions after each upload. When a post goes viral, the rendition fleet saturates, upload requests time out, and some uploads are never processed. The company needs a redesign that guarantees no upload job is lost and handles unpredictable spikes MOST cost-effectively. What should a solutions architect recommend?** Choose one.

- **a.** Place an Amazon SQS queue between the web tier and the rendition fleet, and run the fleet in an Auto Scaling group that scales on the queue backlog.
- **b.** Publish each upload job to an Amazon SNS topic and subscribe the rendition instances as HTTP endpoints.
- **c.** Provision the rendition fleet with enough EC2 instances to handle the largest expected spike at all times.
- **d.** Put an Application Load Balancer in front of the rendition fleet and increase the web tier request timeout.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The queue durably stores every job until a worker deletes it after success, so nothing is lost, and workers scale out on backlog during a spike and back in afterward, so the company pays only for capacity it uses.
- **b** ❌ SNS pushes at its own pace instead of letting saturated workers pull at theirs, and once delivery retries to an overwhelmed endpoint are exhausted the message is gone; it is not a durable consumer-paced buffer.
- **c** ❌ Static peak provisioning keeps the synchronous coupling, still drops work if a spike exceeds the estimate, and pays for idle capacity around the clock, failing the cost qualifier.
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

## Question 8

*Design Resilient Architectures*

**An application already uses an SQS standard queue between its API tier and a worker fleet that is fixed at six EC2 instances. During ingest bursts the backlog grows to hundreds of thousands of messages and takes hours to drain, yet the fleet sits mostly idle between bursts. The bursts do not occur at predictable times. Which solution drains bursts quickly with the LEAST operational overhead?** Choose one.

- **a.** Move the workers into an Auto Scaling group with a target tracking policy based on the queue backlog per instance.
- **b.** Convert the standard queue to a FIFO queue to speed up message processing.
- **c.** Create a CloudWatch alarm on queue depth that pages an operator to resize the fleet.
- **d.** Configure scheduled scaling to add worker instances during expected burst windows.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Target tracking on backlog per instance automatically adds workers as the queue deepens and removes them as it drains, with no schedules to maintain and no human action.
- **b** ❌ FIFO adds ordering and deduplication guarantees at the price of capped throughput; it adds no processing capacity and would make a large backlog drain slower, not faster.
- **c** ❌ Manual resizing works but is the definition of operational overhead: a human must react to every burst, and drain time depends on response time.
- **d** ❌ Scheduled scaling only helps when load follows a known clock; the stem says bursts are unpredictable, so schedules will miss bursts and waste capacity at the wrong times.

**The concept.** A queue only buffers work; draining a backlog quickly requires the consumer tier to scale on the queue itself. Target tracking on backlog per instance is the standard, lowest-touch policy for queue-driven workers.

**Why this is correct.** Target tracking is the winner because it needs a metric and a target, and AWS computes every scaling adjustment from there: as ApproximateNumberOfMessagesVisible grows, backlog per instance rises above target and the group scales out; when the queue drains, it scales back in. That is fully automatic for unpredictable bursts, satisfying LEAST operational overhead. Scheduled scaling fails the scenario premise directly because there is no predictable window to schedule against. The alarm-plus-operator option functions but converts an automation problem into a staffing problem, maximizing rather than minimizing overhead. The FIFO conversion is the precise-fact trap: FIFO queues trade throughput for ordering guarantees, so they add zero capacity and constrain the very throughput a large backlog needs.

**How to reason it out**

1. Recognize that the buffer already exists; the bottleneck is a fixed-size consumer fleet.
2. Eliminate schedule-based answers because the stem states bursts are unpredictable.
3. Eliminate manual intervention because the qualifier demands the least operational overhead.
4. Choose target tracking on backlog per instance so the worker Auto Scaling group sizes itself to the queue automatically.
5. Confirm FIFO is a red herring: it changes delivery semantics, not capacity.

> **Exam tip:** Drain unpredictable SQS backlogs with a target tracking policy on backlog per instance, not schedules, pages, or queue-type changes.

</details>

📖 Learn this topic: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 9

*Design Resilient Architectures*

**A company runs a three-tier application in a single Region and must be able to recover it in a second Region after a regional outage. The business has set an RTO of 60 minutes and an RPO of 5 minutes, and has directed the architecture team to choose the cheapest disaster recovery strategy that meets both objectives. Which strategy should the team select?** Choose one.

- **a.** Run the full stack at production capacity in both Regions with traffic served from both.
- **b.** Run a scaled-down but fully functional copy of the whole stack in the second Region and scale it up during a failover.
- **c.** Continuously replicate the database to the second Region and keep current AMIs and launch templates ready, launching the application tier only during a failover.
- **d.** Copy nightly database snapshots and AMIs to the second Region and rebuild the stack with CloudFormation after a disaster.

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Active-active meets the objectives by the widest margin but roughly doubles infrastructure cost, failing the cheapest-that-meets instruction most severely.
- **b** ❌ Warm standby also meets both objectives but continuously pays for a running application tier the 60-minute RTO does not require.
- **c** ✅ Pilot light meets the numbers at the lowest cost: continuous replication satisfies the 5-minute RPO, and launching pre-staged compute fits an RTO of tens of minutes, inside the 60-minute target.
- **d** ❌ Nightly snapshots allow up to 24 hours of data loss, far beyond the 5-minute RPO, and a from-scratch rebuild typically runs past the 60-minute RTO as well.

**The concept.** DR strategy selection is a ladder of cost against recovery speed: extract the RTO and RPO from the scenario first, then choose the cheapest strategy whose recovery characteristics satisfy both numbers.

**Why this is correct.** The two numbers do the elimination. An RPO of 5 minutes requires continuous data replication, which removes backup and restore immediately, since its RPO equals backup frequency (here, up to a day) and its rebuild-everything RTO is measured in hours. The remaining three strategies all meet the numbers: pilot light recovers in tens of minutes because the data layer is already live and only compute must launch; warm standby recovers in minutes; active-active in seconds. The instruction cheapest that meets both objectives then selects the lowest surviving rung, pilot light, which runs almost no idle compute. Warm standby and active-active are the over-delivery distractors: both work, both cost more, and the 60-minute RTO gives no reason to pay for either.

**How to reason it out**

1. Extract the objectives before reading options: RTO 60 minutes, RPO 5 minutes.
2. Apply the RPO: 5 minutes demands continuous replication, eliminating nightly backup and restore.
3. Apply the RTO: pilot light's tens-of-minutes recovery fits inside 60 minutes, so it qualifies alongside warm standby and active-active.
4. Apply the cost rule: of the strategies that meet both numbers, pick the cheapest rung, which is pilot light.

> **Exam tip:** Read the RTO and RPO first, eliminate under-delivering strategies, then take the cheapest rung that still meets both numbers.

</details>

📖 Learn this topic: [AWS High Availability and Disaster Recovery: RTO, RPO, and the Four DR Levels](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-availability-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 10

*Design Resilient Architectures*

**An order-intake API calls a downstream fulfillment service synchronously for every order. During flash promotions the fulfillment service is overwhelmed, calls fail, and orders are permanently lost. The company requires that no accepted order is ever lost and that fulfillment capacity grows automatically with demand. Which combination of steps should a solutions architect take? (Select TWO.)** Choose 2.

- **a.** Publish orders to an Amazon SNS topic and subscribe the fulfillment instances as HTTP endpoints.
- **b.** Configure the API to retry failed fulfillment calls with exponential backoff.
- **c.** Migrate the fulfillment database to a larger instance class.
- **d.** Change the API to send each accepted order to an Amazon SQS queue instead of calling the fulfillment service directly.
- **e.** Run the fulfillment service in an Auto Scaling group that scales based on the depth of the order queue.

<details>
<summary>Show answer and explanation</summary>

**Answer: d, e**

- **a** ❌ SNS push delivery retries for a while and then drops undeliverable messages; overwhelmed endpoints can still lose orders because there is no consumer-paced durable buffer.
- **b** ❌ Retries help transient errors, but the order exists only in the API instance memory while retrying; sustained saturation or an API crash mid-retry still loses it.
- **c** ❌ This guesses at a bottleneck the stem never identifies and does nothing to decouple the tiers or prevent loss when the service itself saturates.
- **d** ✅ The queue durably persists every accepted order until fulfillment processes and deletes it, so a saturated or failed fulfillment service can no longer cause loss.
- **e** ✅ Scaling on queue depth grows fulfillment capacity exactly when a backlog forms and shrinks it afterward, meeting the automatic-growth requirement.

**The concept.** Guaranteeing no lost work between a producer and an overwhelmed consumer requires two things together: a durable buffer that owns each message until it is processed, and a consumer tier that scales on the buffer's depth.

**Why this is correct.** The queue and the queue-driven Auto Scaling group are two halves of one design. SQS satisfies the no-loss requirement because a message survives consumer failure and reappears after the visibility timeout until a worker deletes it post-success. Scaling fulfillment on queue depth satisfies the elastic-capacity requirement without human action. SNS fails the no-loss requirement on its own: it is push-based, retains nothing for consumer-paced retrieval, and exhausts retries against a saturated endpoint. Client-side exponential backoff keeps the tight coupling; the order lives in volatile memory during retries, so the failure window is narrowed but not closed, and it adds latency to the intake path during exactly the traffic peak. Enlarging the database is vertical scaling aimed at an unstated bottleneck; the stem describes a coupling failure, not a database failure.

**How to reason it out**

1. Restate the two hard requirements: zero lost orders and automatic capacity growth.
2. Map zero loss to durable buffering: only SQS holds an order independently of both the API and fulfillment being healthy.
3. Map automatic growth to a scaling signal: queue depth is the direct measure of unprocessed fulfillment work.
4. Eliminate SNS push and client retries because both leave a window where the only copy of an order can vanish.
5. Eliminate database resizing because it neither buffers nor scales the failing tier.

> **Exam tip:** No-lost-work plus elastic capacity is always the pair: SQS as the durable buffer, workers scaling on queue depth.

</details>

📖 Learn this topic: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 11

*Design Resilient Architectures*

**An internal document-archive application is used a few times per week. After a regional outage, the business can tolerate up to 24 hours of downtime and up to 24 hours of data loss. A consultant has proposed running a warm standby of the application in a second Region. What should a solutions architect recommend to minimize disaster recovery cost while still meeting the objectives?** Choose one.

- **a.** Replace the proposal with backup and restore: take daily backups, copy them to the second Region, and keep the infrastructure defined as code for on-demand rebuild.
- **b.** Keep the warm standby proposal as designed.
- **c.** Upgrade the proposal to active-active deployments in both Regions.
- **d.** Downgrade the proposal to a pilot light with continuous database replication to the second Region.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Daily cross-Region backups meet the 24-hour RPO, an infrastructure-as-code rebuild fits the 24-hour RTO, and nothing runs in the second Region day to day.
- **b** ❌ A continuously running scaled-down stack delivers minutes-level recovery the business never asked for, paying every month for headroom the 24-hour objectives make unnecessary.
- **c** ❌ Active-active is the most expensive strategy on the ladder and over-delivers both objectives by the widest possible margin.
- **d** ❌ Pilot light still pays for an always-on replicated data layer to achieve an RPO of minutes when the business explicitly tolerates 24 hours of loss.

**The concept.** Over-delivering a DR tier is a cost failure: when the stated RTO and RPO are loose, every strategy above backup and restore pays continuously for recovery speed the business has said it does not need.

**Why this is correct.** Both objectives are 24 hours, the loosest realistic tier, and every strategy on the ladder satisfies them; the differentiator is purely cost. Backup and restore is the only strategy that keeps nothing running in the recovery Region: daily backups copied cross-Region give an RPO of at most 24 hours, and rebuilding from infrastructure-as-code comfortably fits a 24-hour RTO for an internal, rarely used application. The consultant's warm standby is the over-delivery trap made explicit: it works, but its recurring compute and data-layer costs buy a minutes-level recovery no requirement justifies. Pilot light is the subtler version of the same mistake, still paying for continuous replication against a 24-hour RPO. Active-active compounds it to roughly double-run cost. The correct instinct on cost-qualified DR questions is to slide down the ladder until just before an objective breaks.

**How to reason it out**

1. Extract the objectives: RTO 24 hours, RPO 24 hours, with an explicit cost-minimization instruction.
2. Note that all four strategies meet these loose numbers, so the decision is cost alone.
3. Identify what each rung pays for continuously: nothing (backup and restore), a live data layer (pilot light), a running stack (warm standby), full duplication (active-active).
4. Select backup and restore as the only strategy whose steady-state cost matches the modest objectives.

> **Exam tip:** When RTO and RPO are measured in hours or a day, backup and restore wins; every higher rung is paying for speed nobody required.

</details>

📖 Learn this topic: [AWS High Availability and Disaster Recovery: RTO, RPO, and the Four DR Levels](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-availability-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 12

*Design High-Performing Architectures*

**A company is deploying a new relational database on a single EC2 instance. Monitoring of the current environment shows a moderate, steady level of random IOPS that is well within the range a general purpose SSD volume can deliver. Which EBS volume type meets the performance requirement MOST cost-effectively?** Choose one.

- **a.** General Purpose SSD (gp3)
- **b.** Provisioned IOPS SSD (io2)
- **c.** Instance store on a storage-optimized instance
- **d.** Throughput Optimized HDD (st1)

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ gp3 delivers low-latency SSD performance for random database I/O at balanced cost, and the stated requirement sits comfortably within what it can provision.
- **b** ❌ io2 would certainly meet the requirement, but it is the most expensive volume type and the workload does not need sustained extreme IOPS or guaranteed sub-millisecond consistency.
- **c** ❌ Instance store is ephemeral, so the database files would be lost on an instance stop or hardware failure — unacceptable for a system of record.
- **d** ❌ st1 is built for large sequential reads and writes; it performs poorly on the small random I/O pattern of a transactional database.

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

## Question 13

*Design High-Performing Architectures*

**A payments company runs a mission-critical OLTP database on an EC2 instance. The database sustains a very high level of random IOPS around the clock — beyond the maximum a gp3 volume can provision — and the business requires consistently low, sub-millisecond storage latency. Which storage option meets these requirements?** Choose one.

- **a.** An Amazon EFS file system mounted on the instance
- **b.** Provisioned IOPS SSD (io2) with IOPS provisioned to match the measured demand
- **c.** Throughput Optimized HDD (st1)
- **d.** General Purpose SSD (gp3) with maximum provisioned IOPS and throughput

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ EFS is shared NFS file storage; its network file-system latency profile is the wrong shape for latency-sensitive database block I/O.
- **b** ✅ io2 exists precisely for sustained very high IOPS with consistent sub-millisecond latency and higher durability, reaching beyond what gp3 can provision.
- **c** ❌ st1 targets large sequential throughput at low cost; it cannot deliver high random IOPS or sub-millisecond latency.
- **d** ❌ The scenario states the demand exceeds what gp3 can provision, so even a fully tuned gp3 volume cannot meet the requirement.

**The concept.** io2 is the top rung of the EBS ladder, justified only when the workload demands sustained IOPS or latency consistency beyond gp3's reach — which this scenario states explicitly.

**Why this is correct.** Two phrases decide it: the demand is beyond what gp3 can provision, and latency must be consistently sub-millisecond. That is the exact brief of Provisioned IOPS SSD, so io2 sized to the measured demand is the only option that satisfies both. gp3 fails on the stated ceiling — no amount of tuning gets it past its provisioning maximum. st1 fails twice: HDD media cannot serve high random IOPS, and it offers no latency guarantee. EFS fails on access shape: database engines expect a low-latency block device, not a network file system, and NFS round-trips cannot honor a sub-millisecond consistency requirement.

**How to reason it out**

1. Confirm the storage type: a database on EC2 needs durable block storage, so EBS is the family.
2. Eliminate on access pattern: random small I/O rules out HDD types like st1, and file storage like EFS.
3. Compare demand to the gp3 ceiling: the scenario says demand exceeds it, eliminating gp3.
4. Match the remaining requirement — sustained IOPS plus consistent sub-millisecond latency — to io2's purpose.
5. Provision io2 IOPS to the measured demand rather than guessing.

> **Exam tip:** Sustained IOPS beyond gp3's reach plus a latency-consistency requirement is the io2 signature.

</details>

📖 Learn this topic: [High-Performing Storage: S3 vs EBS vs EFS, Volume Types, and Hybrid Options](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 14

*Design High-Performing Architectures*

**A web application runs on an Auto Scaling group of EC2 instances behind an Application Load Balancer. The operations team wants the fleet to hold average CPU utilization near 50 percent as traffic rises and falls, with the LEAST configuration to build and maintain. Which scaling approach should a solutions architect recommend?** Choose one.

- **a.** Step scaling policies with multiple CloudWatch alarm thresholds
- **b.** Predictive scaling based on historical traffic patterns
- **c.** A target tracking scaling policy with average CPU utilization as the metric
- **d.** Scheduled scaling actions sized to expected daily traffic

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Step scaling can achieve the same goal but requires designing and maintaining alarm thresholds and adjustment sizes — more configuration for no added benefit here.
- **b** ❌ Predictive scaling forecasts recurring patterns and is useful for slow-warm-up ramps; for a simple hold-a-metric goal it adds machinery target tracking does not need.
- **c** ✅ Target tracking works like a thermostat: set the metric and the value, and the group adds and removes capacity automatically to hold it — no alarms to design.
- **d** ❌ Scheduled scaling follows a calendar, not load; it cannot hold a utilization target when traffic deviates from the plan.

**The concept.** Target tracking is the default and AWS-recommended scaling policy: pick a metric and a target value, and the Auto Scaling group holds it automatically in both directions.

**Why this is correct.** The requirement is a steady utilization goal with minimal configuration — the target tracking definition. One policy, one metric, one number, and the group self-adjusts as demand moves either way. Step scaling is the workable-but-heavier distractor: it can approximate the same behavior only after someone designs alarm thresholds and per-tier adjustments, then maintains them. Scheduled scaling scales on the clock and simply cannot respond to actual load deviations. Predictive scaling is built for recurring, forecastable ramps where reactive policies lag; deploying its forecasting machinery to hold a simple CPU target fails the LEAST-configuration qualifier.

**How to reason it out**

1. Identify the goal: hold a single metric at a set value as load varies.
2. Match the goal to target tracking, the thermostat-style default AWS recommends.
3. Eliminate step scaling as more configuration for the same outcome.
4. Eliminate scheduled (calendar, not load) and predictive (forecasting machinery unneeded) on fit.

> **Exam tip:** Keep a metric at a value with minimal effort is the target tracking signature — step scaling is the higher-maintenance sibling.

</details>

📖 Learn this topic: [Elastic Compute: Auto Scaling Policies, Lambda Tuning, and Compute Selection](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-elastic-compute-solutions/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 15

*Design High-Performing Architectures*

**A retailer is migrating an order-management application to AWS. The application records each order as a transaction spanning inventory, payment, and shipping tables, and analysts run ad hoc SQL queries with multi-table joins. Which database service meets these requirements?** Choose one.

- **a.** Amazon ElastiCache for Redis
- **b.** Amazon Aurora PostgreSQL
- **c.** Amazon Neptune
- **d.** Amazon DynamoDB

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Redis is an in-memory cache and data-structure store, not a transactional system of record for order data.
- **b** ✅ Multi-table transactions, joins, and ad hoc SQL are the defining relational workload, and Aurora is a managed relational engine that supports all of them.
- **c** ❌ Neptune is a graph database for highly connected data such as social networks or fraud rings, not tabular transactions with SQL joins.
- **d** ❌ DynamoDB is a key-value store designed for known access patterns; it does not support ad hoc multi-table joins, no matter how well it scales.

**The concept.** The first database decision on SAA-C03 is type selection from the access pattern: transactions across tables, joins, and ad hoc SQL all point to a relational engine.

**Why this is correct.** The stem names two hard relational requirements: transactions that span multiple tables and analyst-driven ad hoc SQL with joins. Aurora PostgreSQL is a fully managed relational database that provides ACID transactions and full SQL, so it is the only option that satisfies both. DynamoDB fails because it is a key-value store — access patterns must be designed around keys in advance, and there is no join capability, so ad hoc analytical queries would require exporting the data elsewhere. ElastiCache for Redis is an in-memory cache; it holds transient copies of data and has no transactional, durable, relational semantics suitable for a system of record. Neptune is purpose-built for graph traversals over highly connected data; order tables with joins are not a graph workload.

**How to reason it out**

1. List the access-pattern facts in the stem: multi-table transactions, joins, ad hoc SQL.
2. Map each fact to a database type — all three map to relational.
3. Eliminate DynamoDB because it requires access patterns known in advance and cannot join tables.
4. Eliminate the in-memory and graph options as purpose-built for different patterns.
5. Select the managed relational engine, Aurora PostgreSQL.

> **Exam tip:** Joins, transactions, and ad hoc SQL always mean a relational engine (RDS or Aurora) — no non-relational option survives those words.

</details>

📖 Learn this topic: [High-Performing Databases: Aurora vs RDS, DynamoDB, DAX and ElastiCache](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 16

*Design High-Performing Architectures*

**A news website serves images, video clips, and JavaScript bundles from an origin in us-east-1 to readers worldwide. Page loads are slow outside North America, and the origin servers are strained by repeated requests for the same assets. Which service reduces global latency AND offloads the origin MOST effectively?** Choose one.

- **a.** Amazon Route 53 latency-based routing to the origin
- **b.** A Network Load Balancer with cross-zone load balancing enabled
- **c.** AWS Global Accelerator in front of the origin's load balancer
- **d.** Amazon CloudFront with the website as the origin

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Latency-based routing picks the fastest Region among several deployments; with a single origin Region there is nothing to choose between, and no caching occurs.
- **b** ❌ An NLB distributes traffic across targets inside a Region; it does nothing for global latency or origin request volume.
- **c** ❌ Global Accelerator moves traffic onto the AWS backbone but caches nothing — every request for every image still reaches the origin.
- **d** ✅ CloudFront caches static assets at edge locations, so repeated requests are served near the reader without touching the origin — solving both the latency and the offload problem.

**The concept.** CloudFront is a content delivery network: it caches content at edge locations, which both serves users from nearby edges and offloads repeated requests from the origin.

**Why this is correct.** The stem describes cacheable HTTP content (images, video, JavaScript), global readers with distance-driven latency, and an origin strained by repeated identical requests. Caching at the edge is the only mechanism that addresses both symptoms at once: readers fetch assets from a nearby edge location, and the origin sees each asset requested roughly once per edge rather than once per reader. Global Accelerator is the designed distractor — it also improves global performance by putting traffic on the AWS backbone at the nearest edge, but it caches nothing, so origin load is untouched; it exists for TCP/UDP workloads, static IPs, and fast failover, none of which appear here. Latency-based routing requires multiple Regional deployments to route between, and there is only one origin. An NLB is Regional load distribution, orthogonal to both problems.

**How to reason it out**

1. Classify the content: static, cacheable HTTP assets requested repeatedly.
2. Note the two symptoms: distance-driven latency for global users and origin strain from repeats.
3. Match edge caching to both symptoms — CloudFront serves repeats from edge locations.
4. Eliminate Global Accelerator because it accelerates connections but never caches.
5. Eliminate DNS routing and load balancing as not reducing origin request volume.

> **Exam tip:** Cacheable HTTP content plus 'reduce load on the origin' is CloudFront; Global Accelerator improves paths but forwards every request.

</details>

📖 Learn this topic: [Scalable Networks: CloudFront vs Global Accelerator, Transit Gateway, ALB vs NLB](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-network-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 17

*Design Cost-Optimized Architectures*

**A company generates monthly financial reports in Amazon S3. Each report is downloaded several times during its first 30 days, then accessed a few times per month for the next two years and must load with millisecond latency whenever it is requested. Which storage design meets these requirements MOST cost-effectively?** Choose one.

- **a.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Glacier Instant Retrieval 30 days after creation.
- **b.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Standard-IA 30 days after creation.
- **c.** Keep new reports in S3 Standard and add a lifecycle rule that transitions them to S3 Glacier Flexible Retrieval 30 days after creation.
- **d.** Store all reports in S3 Standard for the full two-year retention period.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Glacier Instant Retrieval is priced for data touched about once a quarter; at a few reads per month its higher retrieval fees erode the storage savings compared with Standard-IA.
- **b** ✅ Standard absorbs the frequent first-month access without retrieval fees, and Standard-IA then provides millisecond access at a much lower storage price for monthly reads, cleanly satisfying its 30-day minimum duration.
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

## Question 18

*Design Cost-Optimized Architectures*

**A data processing job writes intermediate artifacts to Amazon S3. The artifacts are rarely read, and a cleanup process must delete them 14 days after creation. Which storage configuration is MOST cost-effective?** Choose one.

- **a.** Store the artifacts in S3 Glacier Flexible Retrieval with a lifecycle expiration rule that deletes them after 14 days.
- **b.** Store the artifacts in S3 Standard-IA with a lifecycle expiration rule that deletes them after 14 days.
- **c.** Store the artifacts in S3 One Zone-IA with a lifecycle expiration rule that deletes them after 14 days.
- **d.** Store the artifacts in S3 Standard with a lifecycle expiration rule that deletes them after 14 days.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Glacier Flexible Retrieval bills a 90-day minimum storage duration and restores asynchronously, so 14-day data pays for 76 unused days and any read would wait minutes to hours.
- **b** ❌ Standard-IA bills a 30-day minimum storage duration, so deleting at 14 days still pays for the remaining 16 days, making it more expensive than Standard for short-lived data.
- **c** ❌ One Zone-IA carries the same 30-day minimum duration charge as Standard-IA, so the minimum-duration penalty applies even at its lower per-GB rate.
- **d** ✅ Standard has no minimum storage duration, so 14-day data is billed for exactly 14 days; every cheaper-looking class would bill a 30-day or longer minimum.

**The concept.** Minimum storage durations are a veto: Standard-IA and One Zone-IA bill at least 30 days, Glacier Instant and Flexible Retrieval bill 90 days, and Deep Archive bills 180 days. Data deleted before the minimum is charged for the remainder anyway.

**Why this is correct.** The artifacts live only 14 days, which is shorter than every minimum storage duration below Standard. Standard-IA and One Zone-IA would each bill the full 30 days, so the apparent per-GB savings become a surcharge. Glacier Flexible Retrieval is worse: it bills 90 days and cannot serve a synchronous read. Only S3 Standard, which has no minimum duration, charges for exactly the 14 days the data exists, so it is the cheapest class that fits, even though the data is rarely read. Rarity of access does not matter when retention is shorter than the minimums.

**How to reason it out**

1. Note the stated retention: objects are deleted 14 days after creation.
2. Check each candidate class's minimum storage duration against 14 days: IA classes bill 30 days, Glacier classes bill 90 or 180.
3. Eliminate every class whose minimum exceeds the retention, because early deletion is billed for the remaining days.
4. Select S3 Standard, which has no minimum duration, and attach a 14-day lifecycle expiration action.

> **Exam tip:** Short-lived data belongs in S3 Standard no matter how rarely it is read; minimum durations turn cheaper classes into surcharges.

</details>

📖 Learn this topic: [Cost-Optimized Storage: S3 Classes, Lifecycle Policies, and EBS Economics](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

## Question 19

*Design Cost-Optimized Architectures*

**An application runs on EC2 with a steady floor of 10 instances 24/7 year-round, up to 15 additional instances during business hours, and a nightly batch job on a separate fleet that reprocesses safely from checkpoints in S3. Everything currently runs On-Demand. Which purchasing design is MOST cost-effective?** Choose one.

- **a.** Run all three layers on Spot Instances to maximize the discount.
- **b.** Buy Standard Reserved Instances covering all 25 web-tier instances.
- **c.** Keep the entire workload On-Demand for maximum flexibility.
- **d.** Buy a Savings Plan sized to the 10-instance baseline, run the business-hours layer On-Demand behind Auto Scaling, and move the batch fleet to Spot Instances.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ The always-on baseline and business-hours tier cannot tolerate a two-minute reclaim, so Spot under-delivers on availability no matter how cheap it is.
- **b** ❌ Committing to the peak pays reserved rates for 15 instances that idle outside business hours; commitments only pay off on steady utilization.
- **c** ❌ The 10-instance floor has run all year, so paying full price for guaranteed usage forfeits up to 72 percent in commitment savings for no benefit.
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

**A company runs an internal reporting application backed by a MySQL database on a memory-optimized Amazon RDS instance that runs 24/7. Analysts use the application only on weekdays between 9 AM and 6 PM, and the database sits idle overnight and on weekends. The application must retain MySQL compatibility. What should a solutions architect recommend to reduce the database cost MOST cost-effectively?** Choose one.

- **a.** Purchase a 1-year Reserved Instance for the current RDS instance class.
- **b.** Migrate the database to Aurora Serverless v2 with MySQL compatibility.
- **c.** Migrate the data to an Amazon DynamoDB table using on-demand capacity mode.
- **d.** Convert the RDS instance to a Multi-AZ deployment.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A Reserved Instance discounts the hourly rate but still pays for every idle night and weekend hour. A discount on waste is still waste.
- **b** ✅ Aurora Serverless v2 bills per ACU-second, scales capacity with the 9-to-5 usage curve, and pauses when idle, so the idle hours cost almost nothing while MySQL compatibility is preserved.
- **c** ❌ DynamoDB would eliminate idle cost but is not MySQL-compatible, so the required application compatibility is broken. A cheaper option that misses a requirement is wrong.
- **d** ❌ Multi-AZ adds a standby instance and roughly doubles instance cost. It improves availability but does nothing to reduce spend.

**The concept.** RDS bills instance-hours whether or not queries arrive, so a database that is busy only during business hours wastes most of its instance cost. Serverless engines convert that idle time into near-zero cost.

**Why this is correct.** The workload is idle roughly three-quarters of the week, and the requirement pins the engine to MySQL. Aurora Serverless v2 is MySQL-compatible, bills per second of consumed capacity, and scales down to zero and pauses while idle, so the company pays only for the working-hours load. The Reserved Instance is the classic trap: it discounts 24/7 instance-hours but still charges for every idle hour the scenario just described. DynamoDB on-demand also removes idle cost but fails the MySQL compatibility requirement, and Multi-AZ increases cost rather than reducing it.

**How to reason it out**

1. Read the workload shape first: busy weekdays 9-6, idle nights and weekends means idle-heavy.
2. Note the hard requirement: MySQL compatibility, which eliminates any non-relational rewrite.
3. Eliminate options that keep paying for idle hours (Reserved Instance) or increase cost (Multi-AZ).
4. Select Aurora Serverless v2, which keeps MySQL compatibility and only bills for consumed capacity.

> **Exam tip:** Idle-heavy plus a relational-compatibility requirement resolves to Aurora Serverless v2, and a Reserved Instance on an idle workload just locks in the waste.

</details>

📖 Learn this topic: [Cost-Optimized Databases: DynamoDB vs RDS, Aurora Serverless, and Caching](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

---

[← Back to the SAA-C03 study guide](README.md)
