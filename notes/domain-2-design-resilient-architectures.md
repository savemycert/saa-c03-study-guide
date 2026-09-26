# Domain 2: Design Resilient Architectures (26%)

This domain covers two jobs: decoupling components so each one scales and fails on its own, and keeping a workload running (or getting it back) when an instance, an AZ, or a whole Region fails. Most questions describe a fragile design plus a qualifier such as "LEAST operational overhead" or "MOST cost-effective", and ask for the fix.

## 2.1 Design scalable and loosely coupled architectures

- **Spot tight coupling first.** Stem cues: "orders are lost during peak traffic", "times out waiting for a downstream service", "every new consumer needs a code change". The fix adds a managed intermediary (queue, topic, event bus, load balancer, API). Making the coupled parts bigger is the trap.
- **Why decoupling helps:** a failed consumer leaves work waiting in the intermediary rather than dropping it, and each tier scales on its own signal (web tier on requests, workers on queue depth).
- **Stateless is the precondition** for scaling out or replacing instances. Sessions go to ElastiCache or DynamoDB, files to S3 or EFS, pending jobs to a queue.
- **SQS decisions:**
  - **Standard** (default): very high throughput, at-least-once delivery, best-effort order, so consumers must be idempotent.
  - **FIFO**: ordering within a message group plus exactly-once processing, at a throughput cap. Pick it only when the stem requires order or forbids duplicates.
  - **Visibility timeout**: a received message is hidden, not deleted, until the consumer deletes it. "Messages processed twice" means the timeout is shorter than processing time, so raise it. Switching to FIFO does not fix this.
  - **Dead-letter queue**: a redrive policy moves a message out after `maxReceiveCount` failed receives. Cue: "messages that repeatedly fail".
  - **Buffer pattern**: web tier → SQS → worker Auto Scaling group scaling on backlog per instance.
- **SNS** pushes one message to every subscriber (SQS, Lambda, HTTPS, Firehose, email/SMS). It does not hold messages for later replay.
- **Fan-out** = SNS topic with one SQS queue per consumer, so each consumer gets its own buffer, retries, DLQ and scaling. Subscription filter policies deliver only matching messages. FIFO topics feed FIFO queues when order matters.
- **Trap:** several teams polling one shared SQS queue compete for messages, so each team sees only part of the stream.
- **EventBridge** = event bus plus rules that match on any field of the JSON event and route to targets. Signals: content-based routing, AWS service state-change events on the default bus, SaaS partner event sources, cross-account buses, archive and replay, EventBridge Scheduler.
- **Step Functions** = orchestration (a central state machine) as opposed to choreography (services reacting to events). It provides Choice, Parallel and Map states and per-step Retry/Catch with exponential backoff.
  - Beats chaining Lambdas: the whole synchronous chain has to fit in one 15-minute invocation, you pay while functions wait on each other, and nothing records how far an execution got.
  - Callback task tokens handle "wait for manual approval". **Standard** workflows are long-running and auditable with exactly-once execution. **Express** workflows are high-volume, short-lived and at-least-once.
- **API Gateway** gives clients a stable contract so the backend can change. It handles throttling, usage plans and API keys, request validation, IAM, Cognito or Lambda authorizers, and response caching. Direct service integrations (write to SQS, start Step Functions) remove glue Lambdas.

| Compute | Choose when | Ruled out by |
|---|---|---|
| Lambda | Short, event-driven, spiky work; per-request billing | Jobs that can exceed 15 minutes; processes that never exit |
| Fargate | Containers, long-running or over 15 minutes, no instances to manage | Need for GPUs, specific instance types or host daemons |
| ECS on EC2 | Containers needing instance control or Spot/Reserved/Savings Plans pricing | "Least operational overhead" |
| EKS | Kubernetes is a stated requirement (manifests, tooling, portability) | No Kubernetes requirement (ECS is simpler) |
| EC2 | Full OS control, non-containerized legacy, host-bound licensing | Serverless or container requirements |

- **Containerizing a monolith as-is** (into ECS/Fargate) is the low-change first step. Microservices come later. Externalize state as part of the move.
- **Scaling:**
  - Vertical scaling has a ceiling, often needs a restart, and remains a single point of failure. It is usually a distractor.
  - Horizontal scaling = an Auto Scaling group across AZs behind an ELB.
  - Policies: **target tracking** is the default and needs the least configuration. **Step** scaling uses custom thresholds, **scheduled** scaling follows known time cycles, and **predictive** scaling learns recurring patterns.
  - ALB routes at layer 7 (paths, hosts). NLB works at layer 4 with static IPs.
- **Offload reads and origin load:** ElastiCache (or DAX for DynamoDB) for repeated reads and sessions; RDS read replicas (asynchronous, slightly stale) for reporting; CloudFront to cache at the edge and carry dynamic requests over the AWS backbone. Match the lever to the bottleneck the stem names.

📖 Full lesson: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 2.2 Design highly available and/or fault-tolerant architectures

| Concept | Promise | Stem wording | Typical design |
|---|---|---|---|
| High availability | Recovers automatically; a brief blip is acceptable | "remain available", "recover automatically" | Multi-AZ, Auto Scaling, ELB, RDS Multi-AZ |
| Fault tolerance | No interruption at all | "no interruption", "no downtime or data loss" | Active redundancy already running, synchronous replication |
| Disaster recovery | Restore elsewhere after losing the environment (usually a Region) | "survive a Region outage", RTO/RPO figures | One of the four DR strategies plus Route 53 |

- Choosing fault tolerance when HA was enough fails a cost qualifier. Choosing HA when the stem said "no interruption" fails the requirement.
- **Single points of failure, with their fixes:**
  - One EC2 instance: ASG across AZs behind a load balancer. Even min = max = 1 gives self-healing.
  - One AZ: span at least two, because subnets belong to a single AZ.
  - One NAT gateway: put one in each AZ and route each private subnet to its own AZ's gateway. This also avoids cross-AZ charges.
  - State on the instance: move it to ElastiCache, DynamoDB, S3 or EFS.
  - One Region: a SPOF only if the requirements say so.
- **Legacy app that cannot change:** put an ALB with health checks in front, wrap the instance in an ASG, and externalize whatever state can be moved. Prefer the smallest change that removes the named SPOF.
- **RDS availability:**
  - **Multi-AZ (classic instance deployment):** synchronous standby in another AZ, automatic DNS failover, no data loss. The standby **cannot be read**.
  - **Multi-AZ DB cluster:** does have readable standbys. Assume the classic deployment unless the stem names this one.
  - **Read replicas:** asynchronous and readable, same Region or cross-Region. Promotion is manual or scripted, and a replica can lag.
  - **Aurora:** storage replicated across three AZs. Its replicas serve reads and are failover targets.
  - **RDS Proxy:** pools connections, holds them through failover and reconnects to the new primary, and stops Lambda bursts from exhausting connections. Cues: "errors during failover", "too many connections".
- **Route 53:** health checks can test an endpoint directly, track a CloudWatch alarm, or roll several checks into a calculated check. An unhealthy record is no longer returned.
  - Failover = active-passive (primary Region to DR Region, or to a static S3/CloudFront page).
  - Weighted = percentage splits for blue-green and canary releases.
  - Latency = fastest Region. With health checks it also serves active-active designs.
  - Geolocation = compliance or localization, not availability.
  - Long TTLs delay failover because clients keep cached answers.
- **RTO vs RPO:**
  - **RTO** = the longest acceptable outage, set by how much recovery infrastructure already runs.
  - **RPO** = the most data loss you can accept, expressed as time and set by how often data leaves the primary.
  - Method: pull both numbers out of the stem before reading options, discard anything that misses either one, then take the cheapest option left.
- **DR ladder** (cheapest and slowest first): backup and restore (hours) → pilot light (tens of minutes) → warm standby (minutes) → multi-site active-active (near zero). See the comparison table for details.
  - Pilot light vs warm standby: can the recovery Region serve a request right now? Pilot light cannot until compute starts. Warm standby can, at reduced capacity.
  - Multi-AZ is not a DR strategy. Any Multi-AZ-only answer fails a Region-failure requirement.
- **Data protection:**
  - **AWS Backup** runs central, policy-based backup plans across EBS, RDS, DynamoDB, EFS, S3 and more, and can copy backups to other Regions and accounts. An isolated account protects backups from ransomware or a compromised account.
  - EBS snapshots are incremental and can be copied across Regions. An **AMI** is the launchable image that pilot light keeps ready.
  - RDS automated backups let you restore to any moment inside the configured retention period. Manual snapshots persist until you delete them.
  - **Replication protects against disasters and versioning protects against mistakes.** CRR copies deletes too. S3 Versioning (plus MFA Delete) recovers from accidental deletion, and CRR requires versioning on both buckets.
- **Operational readiness:**
  - **Immutable infrastructure:** ship a new AMI or image rather than patching in place. Blue-green shifts traffic by weighted records or the load balancer, and rolling back means shifting traffic back.
  - **Service quotas** apply per account and per Region. Raise them in the DR Region before a failover needs that capacity.
  - **Throttling** (`ThrottlingException`): retry with exponential backoff and jitter, and raise the quota for the long term. Bigger instances do not help.
  - **X-Ray** traces requests end to end and builds a service map that shows which hop adds latency or errors. CloudWatch shows *that* a service is unhealthy. X-Ray pinpoints *which hop* of the request is failing.

📖 Full lesson: [AWS High Availability and Disaster Recovery: RTO, RPO, and the Four DR Levels](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-availability-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

[← Back to the study guide](../README.md)
