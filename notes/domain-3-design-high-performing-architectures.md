# Domain 3: Design High-Performing Architectures (24%)

Questions describe a workload's I/O shape, traffic curve, protocol, or data volume, then add a qualifier. Several options work; the answer fits the stated bottleneck at the lowest cost or effort.

## 3.1 Determine high-performing and/or scalable storage solutions

- **Classify first, tune second.** Decide how the application addresses its data before comparing speeds.
  - Block (EBS): a disk the OS mounts; one instance, one AZ. Cues: "boot volume", "database on EC2".
  - File (EFS, FSx): shared directories with locking, mounted by many clients. Cues: "mount", "POSIX", "shared".
  - Object (S3): whole objects over an HTTP API, effectively unlimited scale. Cues: "static assets", "backups", "data lake".
- **EBS ladder:** SSD types (gp3, io2) are sized for IOPS and random I/O; HDD types (st1, sc1) are sized for sequential throughput and cannot be boot volumes.
  - gp3 is the default. You can raise its IOPS and throughput without growing the volume.
  - io2 only when a requirement exceeds gp3: sustained very high IOPS or consistent sub-millisecond latency. Supports Multi-Attach within one AZ for cluster-aware software only.
- **Instance store:** host-attached NVMe, the fastest EC2 storage, erased on stop, hibernate, terminate, or host failure. Only for caches, scratch, or replicated cluster nodes.
- **EFS:** managed NFS for Linux, mounted across AZs, capacity grows and shrinks on its own. A small file system with heavy traffic that runs out of burst credits needs provisioned throughput. Elastic throughput fits spiky access with no tuning. Lifecycle rules move cold files to Infrequent Access.
- **FSx is chosen by keyword:** Windows, SMB, or Active Directory → FSx for Windows File Server. HPC, ML training, or a fast file system over S3 data → FSx for Lustre (scratch for short, cheap jobs; persistent for long-lived). NetApp → FSx for NetApp ONTAP. ZFS → FSx for OpenZFS.
- **S3 speed tools, matched to the bottleneck:**

| Bottleneck | Tool | Why |
| --- | --- | --- |
| Large objects, slow or failing uploads | Multipart upload | Parts upload in parallel and retry individually |
| Large downloads | Byte-range fetches | Parallel range requests, or fetch only the part you need |
| Distant uploaders, one bucket Region | Transfer Acceleration | Enters AWS at the nearest edge, then rides the backbone |
| Very high request rates on one prefix | Spread keys across prefixes | Request rate scales per prefix |
| Global read-heavy delivery | CloudFront in front of S3 | Serves from the edge and offloads the bucket |

- **Hybrid:** Storage Gateway = ongoing on-premises access with a local cache (File for NFS/SMB over S3, Volume for iSCSI in cached or stored mode, Tape for backup software). DataSync = moving data, one-time or on a schedule.

| Requirement | Service | Why |
| --- | --- | --- |
| Shared Linux files across AZs, unpredictable growth | EFS | Concurrent NFS mounts, no capacity planning |
| Fastest I/O for throwaway data | Instance store | No network hop; durability not needed |
| Cheap large sequential scans | EBS st1 | HDD pricing fits sequential reads |
| Nightly copy of on-premises shares into AWS | DataSync | Built for movement, then it stops |

📖 Full lesson: [High-Performing Storage: S3 vs EBS vs EFS, Volume Types, and Hybrid Options](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-storage/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 3.2 Design high-performing and elastic compute solutions

- **Three signals pick the service:** run duration, trigger type, and the team's appetite for managing servers.

| Workload | Service | Why |
| --- | --- | --- |
| Event-driven work that finishes within 15 minutes | Lambda | Per-event scale-out, no servers |
| Containers, least infrastructure to manage | Fargate (under ECS or EKS) | No hosts to patch or size |
| Kubernetes named as a requirement | EKS | Existing manifests and tooling carry over |
| Thousands of queued jobs running for hours | AWS Batch | Job queues, managed compute, scales to zero; Spot for cost |
| Spark, Hadoop, Hive, or Presto named | EMR | Managed home for big-data frameworks |
| GPUs, custom AMIs, OS agents, host licensing | EC2 | Full control of hardware and OS |

- **Batch vs Lambda vs EMR in two steps:** Can a unit of work exceed 15 minutes? Then Lambda is out. Is a big-data framework named? Then EMR; otherwise Batch.
- **EC2 families follow the scarce resource:** M balanced; T burstable on CPU credits (a slowdown under sustained load means the credits ran out); C for CPU; R and X for memory; I and D for local NVMe I/O; P and G for GPUs. Compute Optimizer gives right-sizing recommendations for instances, Lambda memory, and EBS.
- **Lambda tuning:** memory is the only setting, and CPU scales with it. A slow CPU-bound function gets more memory; the shorter run time can offset the higher rate. Cold starts on a latency-sensitive API call for provisioned concurrency. Work that runs for hours is split up with Step Functions or moved to Batch or Fargate.
- **Containers:** ECS vs EKS is the orchestrator choice (EKS only when Kubernetes is required). Fargate vs EC2 launch type is where tasks run (EC2 only when something needs host control, such as GPUs). "EKS on Fargate" is valid; "Fargate vs EKS" compares different layers.
- **Auto Scaling policies:**

| Traffic shape | Policy |
| --- | --- |
| Hold a metric (CPU, requests per target) at a value | Target tracking (the default) |
| Different adjustments at different alarm thresholds | Step scaling |
| Fixed, known times | Scheduled scaling |
| Recurring cycle with varying height; slow-starting instances | Predictive scaling, paired with target tracking |
| Worker fleet reading an SQS queue | Custom backlog-per-instance metric, tracked to a target |
| Group overshoots while new instances boot | Set warm-up and cooldown correctly |

- **Decouple tiers** with SQS (or SNS fan-out to queues) so each tier scales on its own signal and bursts wait in a queue instead of failing.
- **Other levers:** CloudFront caching removes origin load instead of absorbing it. Hibernation resumes an instance with its RAM intact when warm-up is slow.

📖 Full lesson: [Elastic Compute: Auto Scaling Policies, Lambda Tuning, and Compute Selection](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-elastic-compute-solutions/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 3.3 Determine high-performing database solutions

- **Choose the database type from the access pattern:** joins, transactions, and ad hoc SQL → RDS or Aurora. Known key-based lookups at very large scale → DynamoDB. Sub-millisecond reads → an in-memory cache in front of the database of record.
- **Purpose-built engines by keyword:** graph and relationships → Neptune; time-series and IoT telemetry → Timestream; MongoDB-compatible → DocumentDB.
- **Engine choice:** a same-engine (homogeneous) migration is the easy path; changing engines means converting the schema.
- **Read replicas add read capacity; Multi-AZ adds availability.**
  - Read replica: readable, asynchronous (can lag), can sit in another Region, promoted manually. The application must send reads to it.
  - Multi-AZ: synchronous standby with automatic failover. In the classic instance deployment the standby serves no reads.
- **Aurora:** compute is separate from a shared storage volume spread over three AZs. Up to 15 replicas read that same storage with little lag, behind one reader endpoint, and they double as failover targets. Global Database gives remote Regions local reads plus a cross-Region DR path. Serverless v2 adjusts capacity for variable or intermittent load.
- **DynamoDB capacity:** on-demand for unknown or spiky traffic; provisioned plus auto scaling once traffic is steady, because it is cheaper there. Tables can change modes.
- **Hot partitions:** throttling while consumed capacity sits well below provisioned capacity means requests are piling onto one key. Fix the key (higher cardinality, composite keys, or write sharding with a suffix). More capacity does not help.
- **Indexes:** a GSI uses a different partition key, can be added at any time, has its own capacity, and is eventually consistent. An LSI keeps the partition key with an alternate sort key, must be defined at table creation, and supports strongly consistent reads. The usual answer is a GSI.
- **Caching strategy:** lazy loading caches only what is read but can serve stale data. Write-through keeps the cache current but adds work to every write. A TTL limits staleness in both.
- **RDS Proxy** pools connections when Lambda or other highly concurrent clients exhaust database connections. A larger instance only raises the same limit.

| Requirement | Service | Why |
| --- | --- | --- |
| Hourly heavy reporting slows production | Read replica | Varied queries would miss a cache |
| Same pages read constantly | ElastiCache | Repeated identical reads are what caches absorb |
| Microsecond DynamoDB reads, minimal code change | DAX | API-compatible cache for DynamoDB |
| RDS disk I/O saturated | Provisioned IOPS storage | Fixes the measured bottleneck |
| Write spikes overwhelm the database | SQS buffer before the writer | Writes land at a steady rate |
| Strongly consistent DynamoDB reads are too costly | Eventually consistent reads | Strong reads consume twice the capacity |

📖 Full lesson: [High-Performing Databases: Aurora vs RDS, DynamoDB, DAX and ElastiCache](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-high-performance-databases/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 3.4 Determine high-performing and/or scalable network architectures

- **Edge:** CloudFront caches and accelerates HTTP/HTTPS, including dynamic content. Global Accelerator caches nothing; it provides two static anycast IPs, carries TCP and UDP, and fails over between Regions quickly without waiting on DNS.
- **Load balancers:**
  - ALB (layer 7): path and host routing, WebSockets, HTTP/2, Lambda targets.
  - NLB (layer 4): TCP/UDP, very low latency at high scale, a static IP per AZ or an Elastic IP, client source-IP preservation, TLS passthrough. Cross-zone load balancing is off by default.
  - GWLB: sits inline in front of a fleet of third-party firewalls or inspection appliances.
- **Address planning:** a VPC CIDR ranges from /16 to /28. Size for future growth, and never overlap with any network you may connect later, because overlap blocks peering and hybrid routing. Build public, private, and database subnet tiers in each AZ. Every subnet loses five reserved addresses, and every ENI uses an IP. If the VPC runs out, add a secondary CIDR.
- **Placement groups:** cluster = tightly packed inside a single AZ, minimum latency (HPC); spread = separate hardware, at most seven instances per AZ (a few critical instances); partition = groups that share no hardware (Kafka, Hadoop, Cassandra). ENA and EFA add network performance for HPC.
- **VPC connectivity:**

| Requirement | Service | Why |
| --- | --- | --- |
| Two or three VPCs, stable relationships | VPC peering | Direct path, no hub; not transitive |
| Many VPCs plus VPN or Direct Connect | Transit Gateway | Transitive hub routing with route-table segmentation |
| One service shared with many VPCs, CIDRs may overlap | PrivateLink | Interface endpoints to an NLB-fronted service, no network merge |
| Hybrid link needed quickly and cheaply, encrypted | Site-to-Site VPN | IPsec over the internet; about 1.25 Gbps per tunnel |
| Consistent latency for large steady transfers | Direct Connect | Dedicated circuit; weeks to provision; not encrypted by default |
| Hybrid link that survives a circuit failure | Direct Connect + VPN backup | Removes the single point of failure |

- **Global routing:** with latency-based routing, Route 53 answers each user with the Region that responds fastest for them. Geolocation routing is for location rules such as compliance, not speed. Add static IPs or instant failover and the answer becomes Global Accelerator.
- **Local placement:** same-AZ placement cuts latency and cross-AZ charges; multi-AZ wins when availability is the priority.

📖 Full lesson: [Scalable Networks: CloudFront vs Global Accelerator, Transit Gateway, ALB vs NLB](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-scalable-network-architectures/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

## 3.5 Determine high-performing data ingestion and transformation solutions

- **Streaming:**
  - Kinesis Data Streams: you write the consumers. Default retention is 24 hours, extendable to 365 days, and consumers can re-read it. Records with the same partition key share a shard and keep their order. On-demand capacity mode removes shard sizing for unpredictable traffic.
  - Amazon Data Firehose (formerly Kinesis Data Firehose): managed delivery to S3, Redshift, OpenSearch, or HTTP endpoints. It buffers, so delivery is near-real-time. It can transform with an inline Lambda and convert JSON to Parquet or ORC. No replay and no custom consumers.
  - Amazon MSK: pick it when Kafka is named.
  - Common combination: Data Streams feeds real-time consumers while Firehose archives the same stream to S3.
- **Moving files:** DataSync for migrations or scheduled copies; Storage Gateway when on-premises apps keep accessing the data; Transfer Family for partner SFTP/FTPS/FTP into S3 or EFS; Snow family for offline bulk transfer.
- **Data lake:** S3 stores the data and the Glue Data Catalog describes it. Lake Formation grants database-, table-, and column-level access in one place and enforces it across Athena, Redshift Spectrum, Glue, and EMR.
- **Glue:** crawlers infer schemas and partitions into the Data Catalog, which Athena, EMR, and Redshift Spectrum share. Glue jobs run serverless Spark or Python ETL, such as CSV to Parquet.
- **Querying:**

| Requirement | Service | Why |
| --- | --- | --- |
| Occasional ad hoc SQL on S3, nothing to run | Athena | Serverless, billed per data scanned |
| Constant heavy BI, complex joins, many analysts | Redshift | Loaded columnar warehouse; Spectrum reaches S3 |
| Custom Spark or Hadoop code | EMR | Framework-level control, highest overhead |
| Routine transformation, least overhead | Glue job | No cluster to manage |
| Dashboards for business users | QuickSight | Managed BI over Athena, Redshift, S3 |

- **Athena cost and speed fix:** convert to compressed columnar Parquet or ORC and partition on the filter columns, typically date. Existing data goes through a Glue job; new streams can be converted by Firehose. Recrawl afterward. Moving to Redshift or adding EMR nodes does not fix a poor data layout.
- **Pipeline questions:** check every stage. Wrong chains usually hide one bad link, such as Data Streams with no consumer or EMR for a simple conversion.

📖 Full lesson: [Data Ingestion & Transformation: Kinesis, Glue, Athena, and Lake Formation](https://www.savemycert.com/revision/aws-solutions-architect-associate/aws-data-ingestion-transformation/?utm_source=github&utm_medium=readme&utm_campaign=saa-c03-study-guide)

[← Back to the study guide](../README.md)
