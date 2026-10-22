# SAA-C03 Study Guide: AWS Certified Solutions Architect – Associate

A free, open study guide for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam: revision notes for every domain, side-by-side comparisons of commonly confused services, a glossary, 20 worked sample questions, and the official syllabus as a checklist with a full free lesson for every topic.

Maintained by [SaveMyCert](https://www.savemycert.com/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide), where you can read every lesson free, [practice with explained questions](https://www.savemycert.com/practice/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide) and [take timed mock exams](https://www.savemycert.com/mocks/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [Exam domains](#exam-domains)
- [What is in this repo](#what-is-in-this-repo)
- [Syllabus checklist](#syllabus-checklist)
  - [Domain 1: Design Secure Architectures](#domain-1-design-secure-architectures)
  - [Domain 2: Design Resilient Architectures](#domain-2-design-resilient-architectures)
  - [Domain 3: Design High-Performing Architectures](#domain-3-design-high-performing-architectures)
  - [Domain 4: Design Cost-Optimized Architectures](#domain-4-design-cost-optimized-architectures)
- [How to study for SAA-C03](#how-to-study-for-saa-c03)
- [Sample questions](sample-questions.md)
- [Free resources](#free-resources)

## Exam at a glance

| | |
|---|---|
| Exam code | SAA-C03 |
| Level | Associate |
| Questions | 65 |
| Time limit | 130 min |
| Passing score | 720 / 1000 |
| Format | Multiple choice & response |
| Exam fee | $150 |
| Valid for | 3 years |

Exam details change. Always confirm them in the official [AWS Certified Solutions Architect – Associate (SAA-C03) exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html) from Amazon Web Services.

## Exam domains

| # | Domain | Weight | Topics |
|---|---|---|---|
| 1 | [Design Secure Architectures](#domain-1-design-secure-architectures) | 30% | 3 |
| 2 | [Design Resilient Architectures](#domain-2-design-resilient-architectures) | 26% | 2 |
| 3 | [Design High-Performing Architectures](#domain-3-design-high-performing-architectures) | 24% | 5 |
| 4 | [Design Cost-Optimized Architectures](#domain-4-design-cost-optimized-architectures) | 20% | 4 |

That is 4 domains and 14 topics. Spend your time in proportion to the weights: the heaviest domain decides more of your score than the lightest.

## What is in this repo

| File | What it gives you |
|---|---|
| [Domain 1 notes](notes/domain-1-design-secure-architectures.md) | Design Secure Architectures: condensed revision notes per topic |
| [Domain 2 notes](notes/domain-2-design-resilient-architectures.md) | Design Resilient Architectures: condensed revision notes per topic |
| [Domain 3 notes](notes/domain-3-design-high-performing-architectures.md) | Design High-Performing Architectures: condensed revision notes per topic |
| [Domain 4 notes](notes/domain-4-design-cost-optimized-architectures.md) | Design Cost-Optimized Architectures: condensed revision notes per topic |
| [Commonly confused services](comparisons.md) | Side-by-side tables of the services questions set against each other |
| [Glossary](glossary.md) | Every in-scope term and service in one sentence |
| [Sample questions](sample-questions.md) | 20 worked questions with answers and reasoning |
| [Exam-day guide](exam-day-guide.md) | Booking, testing options, scoring, results and retakes |

## Syllabus checklist

Tick each topic off once you can explain it without notes. The "Must know" facts are the ones questions turn on. Each lesson link goes to the complete, free lesson.

### Domain 1: Design Secure Architectures

**Weight: 30%.** Secure access to AWS resources, secure workloads and applications, and the right data-security controls.

📝 Revision notes: [Domain 1: Design Secure Architectures](notes/domain-1-design-secure-architectures.md)

- [ ] **1.1 Design secure access to AWS resources**
  <br>Multi-account access control with AWS Organizations, Control Tower, and SCPs; federated and role-based access (IAM, IAM Identity Center, STS, role switching, cross-account roles); least privilege, root-user hardening and MFA, resource policies, directory federation, and the shared responsibility model.
  - 📖 Lesson: [Designing Secure Access: IAM Best Practices, Roles, and AWS Organizations](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-secure-access-iam-organizations/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: SCPs and RCPs never grant permissions — they cap the maximum; effective access is the intersection of SCPs, RCPs, boundaries, and IAM policies, and an explicit deny always wins.
  - Must know: SCPs bind your principals (including member root users) wherever the resource lives, but never the management account or service-linked roles; RCPs bind your resources against any caller, including outsiders.
- [ ] **1.2 Design secure workloads and applications**
  <br>VPC security architecture: security groups, network ACLs, route tables, NAT gateways, and public/private subnet segmentation; application-security services (Cognito, GuardDuty, Macie, Shield, WAF, Secrets Manager); securing external connections (VPN, Direct Connect); external threat vectors (DDoS, SQL injection); service endpoints and credential security.
  - 📖 Lesson: [Securing Workloads: VPC Design, Security Groups vs NACLs, WAF and Shield](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-vpc-security-secure-workloads/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: A route table defines the subnet: internet gateway route = public, NAT route = private, neither = isolated; only internet-facing load balancers and zonal NAT gateways belong in public subnets.
  - Must know: Zonal NAT gateways sit in a public subnet, one per AZ; a regional NAT gateway needs no public subnet and spans AZs; outbound-only IPv6 uses an egress-only internet gateway.
- [ ] **1.3 Determine appropriate data security controls**
  <br>Encryption at rest with AWS KMS and in transit with ACM/TLS; key access policies, key rotation, and certificate renewal; data access and governance, classification, retention, and recovery; backups, replication, and data lifecycle/protection policies that meet compliance requirements.
  - 📖 Lesson: [AWS Data Security Controls: KMS Encryption, S3 Encryption, and Key Management](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-data-security-encryption-kms/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Requirement mentions editing the key policy, independent decrypt control, disabling the key, a custom rotation schedule or cross-account use → customer managed KMS key; AWS managed keys allow none of those, and their requests are still billed.
  - Must know: A key policy is the root of KMS access: IAM policies work only when it delegates to the account; name a role directly to keep IAM administrators out; split key administrators from key users; use kms:ViaService to limit use to one service; grants delegate without editing the policy and last until revoked or retired.

### Domain 2: Design Resilient Architectures

**Weight: 26%.** Scalable, loosely coupled designs and highly available, fault-tolerant architectures measured against RTO and RPO.

📝 Revision notes: [Domain 2: Design Resilient Architectures](notes/domain-2-design-resilient-architectures.md)

- [ ] **2.1 Design scalable and loosely coupled architectures**
  <br>Event-driven, microservice, and multi-tier designs; decoupling with SQS and pub/sub messaging, API Gateway, and Step Functions orchestration; when to choose serverless (Lambda, Fargate) vs containers (ECS, EKS) and migrating apps into containers; horizontal vs vertical scaling, load balancing (ALB), caching, edge accelerators (CDN), read replicas, and storage types (object, file, block).
  - 📖 Lesson: [Scalable, Loosely Coupled Architectures: SQS, SNS, EventBridge, Step Functions](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-loosely-coupled-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: A queue between tiers turns a traffic spike into a backlog: SQS plus a worker tier scaling on backlog per instance is the default spike-absorption redesign.
  - Must know: SQS FIFO orders messages per message group (set the group ID to the entity whose order matters) and deduplicates sends within five minutes by deduplication ID; it does not prevent redelivery after a visibility timeout.
- [ ] **2.2 Design highly available and/or fault-tolerant architectures**
  <br>Multi-AZ and multi-Region design with Regions, AZs, and Route 53; disaster-recovery strategies (backup and restore, pilot light, warm standby, active-active failover) against RTO/RPO; mitigating single points of failure; failover and distributed design patterns, immutable infrastructure, RDS Proxy; data durability strategies, service quotas and throttling, workload visibility with X-Ray, and improving the reliability of legacy applications.
  - 📖 Lesson: [AWS High Availability and Disaster Recovery: RTO, RPO, and the Four DR Levels](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-availability-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: High availability recovers quickly and may blip; fault tolerance never interrupts. A running hot standby still interrupts during switch-over; only load already shared across active instances does not.
  - Must know: Static stability: to keep N instances after losing one of Z zones, run N × Z/(Z−1) spread evenly (6 needed = 9 across three zones or 12 across two).

### Domain 3: Design High-Performing Architectures

**Weight: 24%.** Matching storage, compute, database, network, and data-pipeline services to performance and scale requirements.

📝 Revision notes: [Domain 3: Design High-Performing Architectures](notes/domain-3-design-high-performing-architectures.md)

- [ ] **3.1 Determine high-performing and/or scalable storage solutions**
  <br>Matching object, file, and block storage (S3, EFS, EBS) and their characteristics to performance demands; hybrid storage solutions; selecting configurations that meet requirements now and scale for future needs.
  - 📖 Lesson: [High-Performing Storage: S3 vs EBS vs EFS, Volume Types, and Hybrid Options](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Classify by access pattern first: mounted raw disk = EBS (block), shared directories across instances = EFS/FSx (file), API-accessed web-scale data = S3 (object).
  - Must know: EBS ladder: gp3 by default (up to 80,000 IOPS and 2,000 MiB/s, single-digit-millisecond latency); io2 Block Express when IOPS exceed that or latency must be sub-millisecond; st1 for cheap sequential throughput; sc1 for cold data that must stay mounted. HDDs never boot.
- [ ] **3.2 Design high-performing and elastic compute solutions**
  <br>Compute services with appropriate use cases (EC2 instance types, AWS Batch, EMR, Fargate); serverless patterns (Lambda, Fargate) and container orchestration (ECS, EKS); EC2 Auto Scaling and AWS Auto Scaling — the metrics and conditions that trigger scaling actions; decoupling workloads so components scale independently; resource sizing (e.g. Lambda memory).
  - 📖 Lesson: [Elastic Compute: Auto Scaling Policies, Lambda Tuning, and Compute Selection](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-elastic-compute-solutions/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Pick the compute service by duration, trigger, and management appetite: event-driven and under 15 minutes = Lambda; containers with least overhead = Fargate; hardware control = EC2; queued heavy jobs = AWS Batch; named big-data frameworks = EMR, or EMR Serverless when no cluster should be chosen or kept running.
  - Must know: Over 15 minutes rules out Lambda — container-image functions included; a named framework (Spark/Hadoop/Hive) means EMR; otherwise batch jobs mean AWS Batch (Spot for retryable work), and a scheduled container job is an ECS task on Fargate started by EventBridge Scheduler.
- [ ] **3.3 Determine high-performing database solutions**
  <br>Relational vs non-relational vs serverless vs in-memory databases (Aurora, DynamoDB, ElastiCache) and engine selection (e.g. MySQL vs PostgreSQL); caching strategies, read replicas, and database proxies/connections; capacity planning (capacity units, instance types, Provisioned IOPS) for read-intensive vs write-intensive access patterns.
  - 📖 Lesson: [High-Performing Databases: Aurora vs RDS, DynamoDB, DAX and ElastiCache](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Pick relational (RDS/Aurora) for ad hoc SQL and unplanned joins; DynamoDB supports ACID transactions, so transactions alone don't rule it out — unplanned query patterns do.
  - Must know: Homogeneous migrations keep the engine; heterogeneous ones need DMS Schema Conversion or AWS SCT for schema and code, plus AWS DMS (full load + CDC) for the data. RDS to Aurora: create an Aurora read replica, then promote.
- [ ] **3.4 Determine high-performing and/or scalable network architectures**
  <br>Edge networking (CloudFront, Global Accelerator); selecting a load-balancing strategy; network design — subnet tiers, routing, IP addressing; topologies for global, hybrid, and multi-tier architectures; connection options (AWS VPN, Direct Connect, PrivateLink) and resource placement for performance.
  - 📖 Lesson: [Scalable Networks: CloudFront vs Global Accelerator, Transit Gateway, ALB vs NLB](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-network-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: CloudFront caches and accelerates HTTP content; Global Accelerator provides two static anycast IPs for TCP/UDP with near-instant cross-Region failover — protocol and static-IP requirements decide between them.
  - Must know: ALB for HTTP routing (path/host rules, Lambda targets); NLB for TCP/UDP at extreme scale, Elastic IPs and client-IP preservation (default on for instance targets, off for IP targets on TCP/TLS); a TCP listener passes TLS through, a TLS listener terminates it; GWLB for inline third-party appliances.
- [ ] **3.5 Determine high-performing data ingestion and transformation solutions**
  <br>Streaming and ingestion (Kinesis), data transfer (DataSync, Storage Gateway, Transfer Family), transformation (AWS Glue, e.g. CSV to Parquet), building and securing data lakes (Lake Formation, S3), analytics and visualization (Athena, Lake Formation, Amazon Quick), and compute for data processing (EMR).
  - 📖 Lesson: [Data Ingestion & Transformation: Kinesis, Glue, Athena, and Lake Formation](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-data-ingestion-transformation/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Kinesis Data Streams = real-time with custom consumers, replay within the retention period (24 hours default, up to 365 days), and per-shard ordering via the partition key.
  - Must know: Each shard takes 1 MB/s or 1,000 records/s of writes and serves 2 MB/s of reads; one partition key always maps to one shard, so a hot key needs a finer-grained key — not more shards or on-demand mode.

### Domain 4: Design Cost-Optimized Architectures

**Weight: 20%.** Cost-optimized storage, compute, database, and network designs, plus the cost-visibility tooling to govern spend.

📝 Revision notes: [Domain 4: Design Cost-Optimized Architectures](notes/domain-4-design-cost-optimized-architectures.md)

- [ ] **4.1 Design cost-optimized storage solutions**
  <br>Cost tooling (Cost Explorer, AWS Budgets, Cost and Usage Report, cost allocation tags, multi-account billing); S3 storage classes, lifecycle tiering, and Requester Pays; block storage volume-type economics (HDD vs SSD); choosing the lowest-cost storage service, size, migration/transfer method, and backup/archival solution; storage auto scaling.
  - 📖 Lesson: [Cost-Optimized Storage: S3 Classes, Lifecycle Policies, and EBS Economics](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Price the whole bill — storage, retrievals, requests and minimum durations (IA 30 days, Glacier Instant/Flexible 90, Deep Archive 180) — before picking a class; the duration trap mainly keeps data that lives days to a couple of weeks in S3 Standard.
  - Must know: Standard-IA vs Glacier Instant Retrieval: both read in milliseconds; data read about monthly favours Standard-IA, data read a few times a year favours Glacier Instant Retrieval's lower storage price despite its higher retrieval fee.
- [ ] **4.2 Design cost-optimized compute solutions**
  <br>Purchasing options (Spot Instances, Reserved Instances, Savings Plans); instance family selection and right-sizing; Lambda vs EC2 vs Fargate cost trade-offs; containers, serverless, and microservices for utilization; hybrid compute options (Outposts); scaling strategies (horizontal vs vertical, EC2 hibernation) and load-balancer choice (ALB layer 7 vs NLB layer 4 vs Gateway LB) against each workload’s availability needs.
  - 📖 Lesson: [Cost-Optimized Compute: Spot vs Reserved vs Savings Plans, and Lambda vs EC2](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-compute/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Layer the purchase mix: commitments sized to the steady baseline, On-Demand behind Auto Scaling for the variable layer, Spot (up to 90% off) for interruptible work — never commit to peak, never put un-interruptible work on Spot.
  - Must know: Depth versus flexibility: Standard RIs and EC2 Instance Savings Plans go up to 72% but lock a family; Compute Savings Plans go up to 66% but follow usage anywhere, including Fargate and Lambda, which no EC2-only instrument covers.
- [ ] **4.3 Design cost-optimized database solutions**
  <br>Cost-effective database services and types (DynamoDB vs RDS, serverless, time series, columnar) and engine selection; backup retention and snapshot-frequency policy design; caching to reduce database spend; capacity planning; homogeneous vs heterogeneous migrations across locations and engines.
  - 📖 Lesson: [Cost-Optimized Databases: DynamoDB vs RDS, Aurora Serverless, and Caching](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Workload shape decides the pricing model: idle-heavy or unpredictable load belongs on serverless (Aurora Serverless v2, DynamoDB on-demand); steady 24/7 load belongs on provisioned capacity with a commitment.
  - Must know: Engine choice is a cost lever: open-source engines carry no license fee, and Aurora's premium buys shared-storage replicas (up to 15 without duplicating storage) — pay it only when you use that.
- [ ] **4.4 Design cost-optimized network architectures**
  <br>NAT gateway cost strategies (single shared vs per-AZ, NAT instance vs NAT gateway); connectivity choices (Direct Connect vs VPN vs internet); minimizing data-transfer costs via routing (Region-to-Region, AZ-to-AZ, private vs public), VPC endpoints, Transit Gateway, and VPC peering; CDN/edge caching needs; throttling strategy and bandwidth allocation (single vs multiple VPNs, Direct Connect speed).
  - 📖 Lesson: [Cost-Optimized Networking: NAT Gateways, VPC Endpoints, and Data Transfer](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-cost-optimized-networking/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
  - Must know: Price the path before the fix: inbound is free, same-AZ private traffic is free, cross-AZ is charged both directions, cross-Region and internet egress are charged — the right answer moves traffic onto a free segment. Paid transfer tools (DataSync, Transfer Acceleration) only buy speed, never lower transfer cost.
  - Must know: Gateway VPC endpoints for S3 and DynamoDB are free and remove NAT processing for that traffic — but only in the route tables they are associated with, and only for the VPC's own Region; interface endpoints for other services (ECR also needs the S3 gateway endpoint for layers) cost less than NAT only at volume.

## How to study for SAA-C03

1. **Read the lesson for each topic** in the checklist above, starting with the heaviest domain. Every lesson is free on the [SAA-C03 revision notes](https://www.savemycert.com/revision/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide).
2. **Practice straight after reading.** Answer [SAA-C03 practice questions](https://www.savemycert.com/practice/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide) on the topic you just read. Each option comes with an explanation of why it is right or wrong.
3. **Review what you got wrong**, re-read that lesson section, and tick the topic off only when you get its questions right.
4. **Take a full-length [SAA-C03 mock exam](https://www.savemycert.com/mocks/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)** under the real time limit. Aim to pass mocks comfortably before you book.
5. **On the last day**, skim the [SAA-C03 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide) instead of starting anything new.

## Sample questions

[sample-questions.md](sample-questions.md) has 20 worked SAA-C03 questions with the answer, why each option is right or wrong, and the reasoning steps.

## Free resources

- [AWS Certified Solutions Architect – Associate (SAA-C03) exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html): the official source (Amazon Web Services)
- [SAA-C03 certification overview](https://www.savemycert.com/certifications/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)
- [SAA-C03 revision notes](https://www.savemycert.com/revision/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide): every lesson, free to read
- [SAA-C03 practice questions](https://www.savemycert.com/practice/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide): with an explanation on every option
- [SAA-C03 mock exams](https://www.savemycert.com/mocks/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide): full-length and timed
- [SAA-C03 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-solutions-architect-associate/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide): the key facts on one page
- [All certification study guides](https://github.com/savemycert/certification-study-guides)

## Contributing

Spotted an error or an out-of-date fact? [Open an issue](../../issues) with the topic and a link to the official source. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and disclaimer

This guide is licensed under [CC BY 4.0](LICENSE). You can reuse and adapt it, including commercially, as long as you credit **SaveMyCert** with a link to https://www.savemycert.com/.

This is an independent study resource. It is not affiliated with or endorsed by Amazon Web Services. AWS Certified Solutions Architect – Associate and SAA-C03 are trademarks of their respective owner. Exam domains and weights are taken from the official exam guide linked above.
