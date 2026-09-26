# Domain 4: Design Cost-Optimized Architectures (20%)

Choosing the cheapest storage, compute, database, and network design that still satisfies every stated requirement. Almost every question says MOST cost-effective: first rule out any option that breaks a requirement, then rank the remaining options by what they pay for.

## 4.1 Design cost-optimized storage solutions

**Cost visibility: match the tool to the job**

| Need in the question | Tool |
| --- | --- |
| Alert before spend crosses a limit (actual or forecasted) | AWS Budgets (the only one of these that alerts) |
| Visualize trends, find which service grew | Cost Explorer |
| Most granular line-item data, queried with Athena | Cost and Usage Report (CUR) in S3 |
| Show which team or project is spending | Cost allocation tags (activate them in Billing) |
| Many accounts, volume tiers reached sooner, shared reservation discounts | Consolidated billing (AWS Organizations) |

**S3 classes: storage price vs access price**

| Class | Fits | Minimum duration | Watch for |
| --- | --- | --- | --- |
| Standard | Frequent or short-lived data | None | Highest per-GB rate, no retrieval fee |
| Standard-IA | About monthly, millisecond reads | 30 days | Retrieval fee, minimum billable object size |
| One Zone-IA | Data you can re-create or that has a copy elsewhere | 30 days | Lost if its single AZ fails |
| Intelligent-Tiering | Access pattern unknown or changing | None | Per-object monitoring fee; very small objects are not tiered |
| Glacier Instant Retrieval | About quarterly, still millisecond | 90 days | Retrieval fee |
| Glacier Flexible Retrieval | Archive, restore in minutes to hours | 90 days | Restores are asynchronous |
| Glacier Deep Archive | Compliance archive, restore in about 12+ hours | 180 days | Cheapest storage, slowest restore |

- **Veto checklist:** a "cheaper" class loses if (1) the data is deleted before that class's minimum duration ends, (2) stated access frequency makes retrieval fees exceed the storage savings, or (3) the objects are tiny.
- **Lifecycle rules:** transitions only move data to colder classes and never back up. A transition to Standard-IA or One Zone-IA needs the object to be at least 30 days old. Expiration actions implement "keep N years, then delete."
- **Versioned buckets:** if storage keeps growing while the live data stays the same size, noncurrent versions are the cause. Fix it with a noncurrent-version transition/expiration rule, not by suspending versioning.
- Add `AbortIncompleteMultipartUpload` to any bucket that receives large uploads, because orphaned parts are billed but do not appear in the object list.
- **Intelligent-Tiering vs lifecycle:** use Intelligent-Tiering for unknown patterns (it has no retrieval fees or minimum-duration charges, and a reheated object returns to the frequent tier). Use lifecycle rules when the pattern is known, because they avoid the monitoring fee.
- **Requester Pays:** downloaders (authenticated AWS principals only) pay request and transfer costs, and the owner pays only for storage. Cue: "share a large dataset without paying for partners' downloads."

**EBS**
- You pay for provisioned size, not for the data stored in it.
- gp3 over gp2: lower per-GB price plus a baseline of 3,000 IOPS / 125 MiB/s that does not depend on volume size. A gp2 volume that is oversized "for performance" should become a smaller gp3 volume, changed online with Elastic Volumes.
- io1/io2 only when gp3 cannot provide the IOPS. st1 for frequently read, large sequential data; sc1 as the cheapest option for cold sequential data. Neither HDD type can be a boot volume or handle random I/O.
- **Four leaks to check:** unattached (`available`) volumes, old snapshots, noncurrent S3 versions, and incomplete multipart uploads.
- Deleting old snapshots is safe (blocks newer ones need are kept). Automate with Data Lifecycle Manager or AWS Backup; archive long-term snapshots to EBS Snapshots Archive.

**Choosing the service and moving data**
- S3 is cheapest per GB for anything accessible through an API. EBS is block storage for one instance. EFS is only worth its higher price when many instances must mount a shared file system. Offering EFS for content that could live in S3 is a common trap.
- EFS lifecycle management moves idle files to Infrequent Access (and optionally to archive). This is the standard answer to "cut EFS cost without changing the app."
- AWS Backup centralizes schedules, cold-storage transitions, and expiration across services. Inbound transfer and same-Region EC2-to-S3 traffic are free.
- DataSync fits when the link can carry the data in time or when sync must continue. Snowball fits when shipping a device is faster than the network. Transfer Family = partners upload over SFTP. Tape Gateway replaces physical tape.

📖 Full lesson: [Cost-Optimized Storage: S3 Classes, Lifecycle Policies, and EBS Economics](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 4.2 Design cost-optimized compute solutions

**Purchase options**

| Option | Commitment | Discount (per lesson) | Key constraint |
| --- | --- | --- | --- |
| On-Demand | None | Baseline | Full price; per-second billing |
| Standard RI | 1 or 3 years, one configuration | Up to 72% | Cannot change instance family |
| Convertible RI | 1 or 3 years | Smaller than Standard | Can be exchanged for another family, size, or OS |
| Compute Savings Plan | $/hour for 1 or 3 years | Up to 72% class | Applies across family, size, Region, OS, tenancy, plus Fargate and Lambda |
| EC2 Instance Savings Plan | $/hour, one family in one Region | Up to 72% | Size and OS can change within the family |
| Spot | None | Up to 90% | Reclaimed with a two-minute notice |

- **Layer by workload shape:** commitments cover only the steady 24/7 floor, On-Demand plus Auto Scaling covers the daily variable layer, and Spot covers interruptible batch work.
- **Wrong answers usually:** commit to peak demand, run everything On-Demand, or put the baseline or a database on Spot.
- A planned change of instance family rules out Standard RIs and EC2 Instance Savings Plans; choose a Compute Savings Plan (or Convertible RIs).
- Usage above a Savings Plan commitment is billed at On-Demand rates. Unused commitments are shared across accounts under consolidated billing.

**Spot, safely**
- Use it only for stateless, checkpointed, or retryable work (queues, CI, rendering, transcoding). Never for databases, stateful sessions, or the only capacity serving users.
- **Mixed instances policy:** an On-Demand base, an On-Demand percentage above that base, and the rest on Spot. List many instance types, sizes, and generations across several AZs (each type-AZ pair is a separate capacity pool). Use the price-capacity-optimized allocation strategy and capacity rebalancing.
- A Spot fleet with one instance type in one AZ is fragile. Answers that mention "bidding strategies" are outdated: you pay the current Spot price, and interruptions depend on capacity, not on bids.

**Right-size first, then discount**
- Compute Optimizer recommends instance types (and Lambda memory settings) from CloudWatch history. Budgets alerts and Cost Explorer analyzes; swapping these roles is a common distractor.
- Match the family to the bottleneck (m general, c compute, r/x memory, i/d storage, g/p GPU). Newer generations and Graviton (where the software is compatible) give better price-performance.
- Buying a commitment for an oversized fleet locks that waste in for the full term.

**Service choice by utilization**

| Pattern | Cheapest fit |
| --- | --- |
| Low, spiky, event-driven, under 15 minutes per run | Lambda (idle costs nothing) |
| Sustained high utilization | EC2 with commitments or Spot |
| Containers without managing a cluster, modest or intermittent load | Fargate |
| Containers at high sustained scale | EC2-backed containers packed densely |


**Other cost levers**
- Scale horizontally; scale-in and scheduled scaling are what save money. Stop dev/test off-hours (EBS still bills).
- Hibernation: RAM is saved to the encrypted EBS root volume. You pay for storage, not compute, while it sleeps. Cue: "resume without rebuilding in-memory state."
- Put CloudFront in front of repeated content so the origin fleet can shrink. Use one ALB with host/path rules instead of one load balancer per service. Use NLB for layer-4 throughput or static IPs.

📖 Full lesson: [Cost-Optimized Compute: Spot vs Reserved vs Savings Plans, and Lambda vs EC2](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-compute/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 4.3 Design cost-optimized database solutions

**How it is billed:** RDS charges per instance-hour whether or not it is busy. DynamoDB charges for requests (or capacity-unit-hours) plus storage. The more idle time a workload has, the more serverless or request-based pricing wins.

| Workload shape | Cheapest fit |
| --- | --- |
| Steady 24/7 relational | Provisioned RDS/Aurora + Reserved Instances |
| Business hours only, or unpredictable relational | Aurora Serverless v2 (per-second ACUs, can pause when idle) |
| Spiky or new key-value traffic | DynamoDB on-demand |
| Steady, high-volume key-value | DynamoDB provisioned + auto scaling |
| Repeated reads, staleness acceptable | Smaller database + ElastiCache (DAX for DynamoDB) |
| Varied or must-be-fresh reads, isolated reporting | Read replica |
| Oracle/SQL Server licensing complaint | SCT + DMS to an open-source-compatible engine |
| Rarely restored, multi-year history | Snapshot export to S3, then lifecycle to Glacier |

- Serverless and on-demand charge a premium per unit, so a steady, busy workload costs more on them. A Reserved Instance on a database that sits idle at night locks in that idle cost.
- Stopping RDS pauses only instance charges, and it restarts itself after seven days: a dev/test tactic.
- **Engines:** MySQL, PostgreSQL, and MariaDB have no license fee. Oracle and SQL Server are License Included or BYOL. Aurora costs more per instance, but up to 15 replicas share one storage volume, while each RDS replica pays for its own full copy of storage. Aurora wins when you need several replicas; plain RDS wins for small single-instance databases.
- **Migrations:** homogeneous = DMS only; heterogeneous = SCT + DMS. The conversion effort is paid once, and the license saving continues every year.
- **Right-size:** if CPU and connections stay low for weeks, move to a smaller class; don't reserve the oversized one. Use burstable t-class for small, occasionally busy databases.
- Use storage auto scaling instead of padding the volume in advance. Provisioned IOPS without a measured, sustained IOPS need is waste; switch to gp3.
- **Caching:** each cache hit is a query the database never runs. DAX is API-compatible and its hits do not consume read capacity.
- **RDS Proxy:** pools connections from high-concurrency clients such as Lambda, so the database can be sized for query load rather than connection count. It has its own hourly charge.
- **Backups:** backup storage up to the size of provisioned storage is included for an active database. Automated retention is 1 to 35 days, so set it to the stated RPO. Manual snapshots never expire and survive instance deletion, so delete stale ones.

📖 Full lesson: [Cost-Optimized Databases: DynamoDB vs RDS, Aurora Serverless, and Caching](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 4.4 Design cost-optimized network architectures

**Price each hop**

| Path | Charged? |
| --- | --- |
| Internet into AWS | No |
| Same AZ over private IPs | No |
| Same AZ over public or Elastic IPs | Yes, so switch to private addressing |
| Cross-AZ in a Region | Yes, in both directions |
| Cross-Region | Yes, rate depends on the Region pair |
| Out to the internet | Yes, tiered per GB |
| AWS origin to CloudFront | No |
| Private subnet to S3/DynamoDB through NAT | NAT hourly + per-GB processing |
| Through a gateway endpoint to S3/DynamoDB | No |

**NAT gateway**
- Two meters: an hourly fee for existing plus a per-GB processing fee, on top of normal transfer charges. Adding or resizing NAT gateways does not reduce the per-GB charge.
- A single shared NAT has one hourly fee, but traffic from other AZs pays cross-AZ charges and every subnet depends on one AZ. Use it for dev/test or cost-first setups.
- One NAT per AZ, each with its own route table, is required when the scenario states AZ-failure tolerance. Removing cross-AZ traffic partly offsets the extra hourly fees.
- A NAT instance is cheapest, but you manage it and it is limited by instance bandwidth. It is wrong whenever "managed" or "highly available" appears.
- **Reducing a NAT bill, in order:** gateway endpoints, then interface endpoints for high-volume services, then check VPC Flow Logs, then consolidate NATs only if availability allows.

**Endpoints**
- Gateway endpoints exist only for S3 and DynamoDB. They are route-table entries with no hourly or per-GB fee, and they keep traffic private. This is the most-tested network cost fix.
- Interface endpoints (PrivateLink) serve other services and are billed per hour per AZ plus per GB. They beat NAT only when traffic volume is high. An interface endpoint for S3 works but costs more than the free gateway endpoint.

**Egress and edge**
- CloudFront reduces origin fetches through caching, and the origin-to-CloudFront transfer is free. It helps only for cacheable content.
- Global Accelerator improves performance and provides static IPs at extra cost. It does not cache and is a distractor in egress-cost questions.

**Hybrid and VPC-to-VPC**

| Choice | Pick when |
| --- | --- |
| Site-to-Site VPN | Low or bursty volume, needed quickly, low fixed cost; about 1.25 Gbps per tunnel |
| Multiple VPNs + ECMP on Transit Gateway | More VPN bandwidth is needed |
| Direct Connect | Large steady volume (lower per-GB rate outweighs fixed port and circuit costs); takes weeks to provision; ports of 1/10/100 Gbps, smaller hosted connections through partners |
| VPN now, Direct Connect later | Connectivity is needed immediately and high volume long term; the VPN stays as failover |
| VPC peering | A few stable VPCs; no attachment or processing fee; non-transitive mesh |
| Transit Gateway | Many or growing VPCs, transitive routing, shared hybrid links; per-attachment hourly + per-GB fees |
| PrivateLink | Exposing one service to many VPCs |

- Keep cross-AZ traffic that provides resilience. Reduce the rest by placing components together, compressing and batching messages, and throttling bulk transfers.
- Reviewing a workload: Cost Explorer finds the growing transfer category, VPC Flow Logs show whose traffic it is; fix the largest first.

📖 Full lesson: [Cost-Optimized Networking: NAT Gateways, VPC Endpoints, and Data Transfer](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-networking/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

[← Back to the study guide](../README.md)
