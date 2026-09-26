# SAA-C03 Commonly Confused Services and Concepts

Side-by-side comparisons of the SAA-C03 services and ideas that exam questions most often set against each other. Each table ends with the one thing to remember.

- [Domain 1: Design Secure Architectures](#domain-1-design-secure-architectures)
- [Domain 2: Design Resilient Architectures](#domain-2-design-resilient-architectures)
- [Domain 3: Design High-Performing Architectures](#domain-3-design-high-performing-architectures)
- [Domain 4: Design Cost-Optimized Architectures](#domain-4-design-cost-optimized-architectures)

## Domain 1: Design Secure Architectures

### SCPs vs IAM Policies vs Permissions Boundaries

| | Service control policy (SCP) | Identity-based IAM policy | Permissions boundary |
|---|---|---|---|
| Attaches to | Organization root, OU, or member account | User, group, or role | One user or role |
| Grants permissions? | No, it only limits the maximum | Yes, the only one of the three that grants | No, it only caps the maximum |
| Root user | Constrains member-account root; never touches the management account | Does not govern root | Does not govern root |
| Use it when | A rule must bind everyone in an account, admins included | A principal needs access to something | Developers may create roles but must not escalate privileges |
| Exam cue | "no one in any account, including administrators" | "grant the team access to" | "delegate IAM safely" |

**Remember:** only an IAM policy can say yes. SCPs and boundaries can only narrow the answer, and an explicit deny anywhere wins.

### AWS KMS vs AWS CloudHSM

| | AWS KMS (customer managed key) | AWS CloudHSM |
|---|---|---|
| Tenancy | Multi-tenant HSM fleet run by AWS | Dedicated single-tenant HSMs in your VPC |
| Key control | You set policy and usage; AWS operates the hardware | Only you; AWS has no access to key material |
| Access control | IAM, key policies, and grants | HSM users you manage on the device |
| Service integration | Native with S3, EBS, RDS, and most services | Through standard libraries (PKCS #11, JCE), or as a KMS custom key store |
| Operational effort | Fully managed | You run the cluster, users, and availability |
| Exam cue | "auditable, centrally controlled keys" | "single-tenant", "dedicated hardware", "provider must never access keys" |

**Remember:** KMS is the default answer. Pick CloudHSM only when the stem explicitly demands dedicated hardware, and use a custom key store when it also needs native service integration.

### Security Groups vs Network ACLs

| | Security group | Network ACL |
|---|---|---|
| Attaches to | A resource's network interface (EC2, RDS, load balancer) | A subnet |
| State | Stateful: replies are allowed automatically | Stateless: replies and ephemeral ports need their own rules |
| Rules | Allow only, all evaluated together | Allow and deny, first match by rule number |
| Best use | Per-resource control and tier-to-tier references by security group ID | Subnet-wide guardrails and blocking a specific IP range |
| Exam cue | "only the app tier may reach the database" | "deny a malicious CIDR", "responses never return" |

**Remember:** an explicit deny means a NACL. For tier-to-tier access, reference the source security group instead of a CIDR.

### Secrets Manager vs Parameter Store

| | AWS Secrets Manager | Systems Manager Parameter Store |
|---|---|---|
| Built for | Secrets such as database passwords and API keys | Configuration values, plus secrets as SecureString |
| Rotation | Managed automatic rotation (native for RDS, Redshift, DocumentDB; Lambda for others) | None built in; you script it yourself |
| Cross-account | Supported through resource policies | Not the tool for it |
| Cost | Charged per secret and per API call | Standard tier is free |
| Exam cue | "automatic rotation" | "most cost-effective", "store configuration" |

**Remember:** both encrypt with KMS and use IAM for access. Rotation and cross-account sharing point to Secrets Manager, and cost points to Parameter Store.

### Cognito User Pools vs Identity Pools

| | User pool | Identity pool |
|---|---|---|
| Job | Authentication: who the user is | Authorization to AWS: what the user can touch |
| Output | JWTs for app sessions and API authorization | Temporary AWS credentials from STS under an IAM role |
| Features | Sign-up, sign-in, MFA, social and SAML/OIDC login; authorizer for API Gateway and ALB | Swaps tokens from a user pool or external IdP; supports unauthenticated guest access |
| Exam cue | "add sign-in to the app", "log in with social accounts" | "mobile app uploads directly to S3", "guest access to AWS resources" |

**Remember:** a user pool signs users into your app, and an identity pool gives them AWS credentials. Real designs often chain the two.

### AWS Shield vs AWS WAF

| | AWS Shield | AWS WAF |
|---|---|---|
| Layer | Network and transport (3/4) | Application (7) |
| Stops | SYN floods, UDP reflection, volumetric DDoS | SQL injection, XSS, bad IPs, abusive request rates |
| Tiers / setup | Standard is automatic and free; Advanced adds a 24/7 response team and cost protection | Managed and custom rules on CloudFront, ALB, API Gateway, or AppSync |
| Exam cue | "DDoS", "attack-driven scaling costs" | "SQL injection", "rate-limit clients" |

**Remember:** Shield does not inspect HTTP, and WAF does not stop a SYN flood. A public web app under both kinds of attack needs both.

## Domain 2: Design Resilient Architectures

### Backup and Restore vs Pilot Light vs Warm Standby vs Multi-Site Active-Active

| | Backup and restore | Pilot light | Warm standby | Multi-site active-active |
|---|---|---|---|---|
| Running in the DR Region | Nothing; only copied backups and snapshots | A live, replicated data layer; compute off or minimal | The full workload at reduced size | The full workload at production size, serving users |
| Can it serve requests before failover? | No | No | Yes, at reduced capacity | Yes, it already does |
| RTO class | Hours | Tens of minutes | Minutes | Near zero |
| RPO class | Backup frequency | Seconds to minutes | Seconds | Near zero |
| Relative cost | Lowest | Low | Higher | Highest |
| Exam cue | "lowest cost", "can tolerate hours of downtime" | "core data replicated", "minimize idle infrastructure" | "smaller copy already running", "recover within minutes" | "no downtime", "users served from both Regions" |

**Remember:** take the cheapest rung that meets both RTO and RPO. Anything faster is over-spending, and anything slower fails the requirement.

### RDS Multi-AZ vs Read Replicas

| | RDS Multi-AZ (classic instance deployment) | Read replica |
|---|---|---|
| Replication | Synchronous | Asynchronous (can lag) |
| Readable? | No, the standby serves no traffic | Yes |
| Failover | Automatic, by repointing the DNS endpoint | Manual or scripted promotion |
| Location | Another AZ in the same Region | Same Region or another Region |
| Use it when | You need in-Region availability with no data loss and no manual steps | You need to offload reads or reporting, or seed a DR Region |
| Exam cue | "AZ failure", "automatic failover", "no data loss" | "offload reporting queries", "read-heavy", "cross-Region copy" |

**Remember:** you cannot send reporting queries to a Multi-AZ standby. If a stem needs both failover and read scaling, the answer uses both features.

### SQS vs SNS vs EventBridge

| | Amazon SQS | Amazon SNS | Amazon EventBridge |
|---|---|---|---|
| Model | Pull queue; each message goes to one consumer | Push pub/sub; every subscriber gets a copy | Event bus; rules match event content and route to targets |
| Holds messages | Yes, a durable buffer until they are processed | No replay; pair it with SQS for durability | Optional archive and replay |
| Filtering | None (consumers receive what is in the queue) | Subscription filter policies | Rules on any field of the JSON event |
| Use it when | Buffering a spike, load leveling, retries for one processing tier | Plain, high-throughput one-to-many delivery | Content-based routing, AWS service events, SaaS partner events, cross-account buses |
| Exam cue | "requests lost during spikes", "process asynchronously" | "several systems must react", "every service gets every message" | "route by fields in the event", "third-party SaaS events" |

**Remember:** SQS delivers each message to one consumer and SNS delivers to all subscribers. When routing depends on what the event says, use EventBridge. For reliable fan-out, subscribe one SQS queue per consumer to an SNS topic.

### SQS Standard vs SQS FIFO

| | Standard queue | FIFO queue |
|---|---|---|
| Throughput | Nearly unlimited | Capped |
| Delivery | At least once; duplicates are possible | Exactly-once processing within the deduplication window |
| Ordering | Best effort | Strict within a message group |
| Consumer requirement | Must be idempotent | Can rely on order and no duplicates |
| Exam cue | "high volume", no ordering requirement | "processed in the order received", "exactly once" |

**Remember:** use a standard queue unless the stem requires ordering or forbids duplicates. Duplicate processing on a standard queue is usually fixed by raising the visibility timeout.

### Route 53 Failover vs Weighted vs Latency vs Geolocation Routing

| | Failover | Weighted | Latency-based | Geolocation |
|---|---|---|---|---|
| Sends traffic to | The primary while healthy, otherwise the secondary | Records in proportion to assigned weights | The Region that responds fastest for the user | A record chosen by the user's location |
| Availability role | Active-passive Region failover | Blue-green and canary traffic shifting | Active-active routing that skips unhealthy Regions when paired with health checks | None; it is for compliance or localization |
| Exam cue | "redirect to the standby Region automatically" | "send a percentage of traffic" | "closest/fastest Region" | "users in a country must see local content" |

**Remember:** failover is for surviving an outage, weighted is for releases, latency is for performance, and geolocation is for rules about where users are. Every policy depends on health checks and on clients re-resolving within the TTL.

### S3 Versioning vs S3 Cross-Region Replication

| | S3 Versioning | S3 Cross-Region Replication (CRR) |
|---|---|---|
| Defends against | Accidental deletion and overwrites | Loss of access to a whole Region |
| How | Keeps every version; a delete adds a delete marker | Asynchronously copies new objects to a bucket in another Region |
| Recovery | Remove the delete marker or restore an earlier version | Serve from the replica bucket |
| Limitation | Every copy stays in the same Region | Deletes and bad writes are replicated too |
| Exam cue | "user accidentally deleted objects" | "data must be available if the Region fails" |

**Remember:** replication protects against disasters and versioning protects against mistakes. CRR also requires versioning on both buckets.

## Domain 3: Design High-Performing Architectures

### EBS gp3 vs io2 vs st1 vs sc1

| | gp3 | io2 | st1 | sc1 |
| --- | --- | --- | --- | --- |
| Media | SSD | SSD | HDD | HDD |
| Built for | Balanced IOPS and throughput, raised independently of size | Sustained high IOPS with consistent sub-millisecond latency | Low-cost sequential throughput | Lowest cost per GB |
| Boot volume? | Yes | Yes | No | No |
| Use it when | Boot disks, app servers, most databases | Critical databases whose needs exceed gp3 | Big data, ETL scratch, log processing | Rarely accessed data where cost comes first |
| Exam cue | "general purpose", "more IOPS without more storage" | "sustained", "latency-sensitive", "mission-critical database" | "sequential", "large files", "throughput" | "infrequently accessed", "lowest cost" |

**Remember:** Start at gp3. Move to io2 only when a stated requirement exceeds it, and to HDD only for sequential access.

### EFS vs FSx for Windows File Server vs FSx for Lustre

| | Amazon EFS | FSx for Windows File Server | FSx for Lustre |
| --- | --- | --- | --- |
| Protocol and clients | NFS, Linux | SMB, Windows | Parallel POSIX file system |
| Signature feature | Mounted across AZs; capacity grows and shrinks on its own | Active Directory integration, NTFS features | Can link to an S3 bucket and present objects as files |
| Use it when | Many Linux instances share the same files | Windows apps, SMB shares, AD users | HPC, ML training, or burst processing of S3 data |
| Exam cue | "shared", "POSIX", "across Availability Zones" | "Windows", "SMB", "Active Directory" | "HPC", "parallel", "high-throughput processing over S3" |

**Remember:** EFS is the Linux default. Any mention of Windows or HPC sends the answer to an FSx variant.

### ElastiCache for Redis vs Memcached vs DAX

| | ElastiCache for Redis | ElastiCache for Memcached | DAX |
| --- | --- | --- | --- |
| Works with | Any data source | Any data source | DynamoDB only |
| Durability and failover | Snapshots, replication, Multi-AZ automatic failover | None; a lost node loses its data | Write-through cache in front of DynamoDB |
| Data model | Rich types (sorted sets, hashes, lists) plus pub/sub | Simple key-value | DynamoDB items, same API |
| Code changes | Application writes the caching logic | Application writes the caching logic | Swap the client; code stays the same |
| Exam cue | "leaderboard", "cache must survive node failure" | "simplest cache", "multi-threaded", "data loss acceptable" | "DynamoDB", "microseconds", "no application changes" |

**Remember:** DAX for DynamoDB, Redis for features and resilience, and Memcached for plain, disposable caching.

### Kinesis Data Streams vs Amazon Data Firehose

| | Kinesis Data Streams | Amazon Data Firehose |
| --- | --- | --- |
| Role | Ordered, retained stream that your consumers read | Managed pipe that delivers to a destination |
| Latency | Real time | Near-real-time (buffers by size or time) |
| Replay | Yes; 24 hours by default, extendable to 365 days | No |
| Who writes code | You build and run the consumers | No consumer code; optional inline Lambda transform |
| Destinations | Whatever your consumers write to | S3, Redshift, OpenSearch, HTTP endpoints |
| Exam cue | "real-time processing", "replay", "ordered per device" | "deliver to S3", "convert to Parquet", "least overhead" |

**Remember:** Data Streams holds data for your code to process; Firehose delivers data somewhere with no code. If Kafka is named, the answer is MSK.

### Athena vs Redshift vs EMR

| | Amazon Athena | Amazon Redshift | Amazon EMR |
| --- | --- | --- | --- |
| Model | Serverless SQL directly on S3 | Data warehouse with loaded columnar storage | Managed Spark/Hadoop cluster |
| You pay for | Data scanned per query | Warehouse capacity | Cluster instances |
| Operational overhead | None | Moderate | Highest |
| Use it when | Ad hoc or occasional queries on the lake | Constant heavy BI with complex joins | Custom big-data code and frameworks |
| Exam cue | "serverless", "query S3 with SQL", "infrequent" | "enterprise dashboards", "hundreds of analysts" | "Apache Spark", "Hadoop", "custom processing" |

**Remember:** Athena for occasional SQL, Redshift for heavy repeated BI, EMR for custom framework code. A slow Athena query needs Parquet and partitions before it needs another service.

### CloudFront vs Global Accelerator

| | Amazon CloudFront | AWS Global Accelerator |
| --- | --- | --- |
| Caching | Yes, at edge locations | None |
| Protocols | HTTP and HTTPS | TCP and UDP |
| Entry point | Distribution domain name | Two static anycast IP addresses |
| Offloads the origin | Yes, cached responses never reach it | No, every connection reaches an endpoint |
| Regional failover | Not its role | Health-check driven, in seconds, no DNS wait |
| Exam cue | "cache images and video", "reduce origin load" | "UDP", "gaming", "VoIP", "allowlist fixed IPs" |

**Remember:** Both reduce latency for global users. The protocol and any static-IP requirement decide which one fits.

### Cluster vs spread vs partition placement groups

| | Cluster | Spread | Partition |
| --- | --- | --- | --- |
| Layout | Packed close together in one AZ | Each instance on separate hardware | Groups of instances that share no hardware with other groups |
| Scale limit | Capacity in one AZ | Seven instances per AZ per group | Up to seven partitions per AZ, many instances each |
| Optimizes | Lowest node-to-node latency, highest throughput | Isolation of a few critical instances | Failure isolation for large distributed systems |
| Trade-off | One AZ holds everything | Too small for big fleets | Isolation is per partition, not per instance |
| Exam cue | "HPC", "tightly coupled" | "must not fail together", small count | "Kafka", "Hadoop", "Cassandra", "hundreds of instances" |

**Remember:** Cluster for speed, spread for a few critical instances, partition for big distributed data systems.

## Domain 4: Design Cost-Optimized Architectures

### Standard-IA vs One Zone-IA vs Intelligent-Tiering vs Glacier Deep Archive

| | Standard-IA | One Zone-IA | Intelligent-Tiering | Glacier Deep Archive |
| --- | --- | --- | --- | --- |
| Access pattern | Known, about monthly | Known, infrequent | Unknown or changing | Once a year or less |
| Read latency | Milliseconds | Milliseconds | Milliseconds (optional archive tiers are asynchronous) | About 12 hours or more |
| Minimum duration | 30 days | 30 days | None | 180 days |
| Retrieval fee | Yes | Yes | No | Yes |
| Resilience | Multiple AZs | One AZ | Multiple AZs | Multiple AZs |
| Hidden cost | Frequent reads | Losing the only copy | Per-object monitoring fee | Early deletion |
| Exam cue | "monthly, immediately available" | "can be regenerated" | "unpredictable access" | "seven-year compliance archive" |

**Remember:** choose the class that matches the stated access frequency, then reject it if the retention period is shorter than its minimum duration or the data has no other copy.

### Compute Savings Plan vs EC2 Instance Savings Plan vs Reserved Instances vs Spot

| | Compute Savings Plan | EC2 Instance Savings Plan | Reserved Instances | Spot |
| --- | --- | --- | --- | --- |
| You commit to | $/hour for 1 or 3 years | $/hour for one family in one Region | An instance configuration for 1 or 3 years | Nothing, but accept reclamation |
| Discount (per lesson) | Up to 72% class | Up to 72% | Up to 72% (Standard); less (Convertible) | Up to 90% |
| Family change allowed | Yes, plus Region, OS, tenancy | No (size and OS yes) | Standard no; Convertible via exchange | Any type you list |
| Covers Fargate/Lambda | Yes | No | No | Not Lambda |
| Right layer | Steady floor that may change | Steady floor on a fixed family | Steady floor; also RDS, ElastiCache | Interruptible batch and queue work |
| Exam cue | "migrate to a new family this year" | "committed to one family" | "database runs 24/7 for three years" | "restartable", "checkpoints" |

**Remember:** size commitments to the steady floor, not the peak, and never put anything that must stay running on Spot.

### NAT gateway vs gateway VPC endpoint vs interface VPC endpoint

| | NAT gateway | Gateway endpoint | Interface endpoint |
| --- | --- | --- | --- |
| What it is | Managed egress from private subnets | Route-table entry | Elastic network interface (PrivateLink) |
| Services | Anything on the internet | S3 and DynamoDB only | Most other AWS services |
| Cost | Hourly + per GB processed | Free | Hourly per AZ + per GB |
| Cheaper than NAT when | Not applicable | Always, for S3/DynamoDB | Traffic volume is high |
| Exam cue | "private subnets need internet" | "NAT processing charges from S3 traffic" | "private access to SQS/ECR/Secrets Manager" |

**Remember:** send S3 and DynamoDB traffic through the free gateway endpoint first. Adding NAT gateways never reduces the per-GB charge.

### Aurora Serverless v2 vs provisioned Aurora/RDS

| | Aurora Serverless v2 | Provisioned + Reserved Instances |
| --- | --- | --- |
| Billing | ACUs per second as load changes | Instance-hours, busy or idle |
| Idle cost | Near zero (can pause; storage still billed) | Full instance rate |
| Sustained high load | More expensive than an equivalent instance | Cheapest, especially reserved |
| Compatibility | Full MySQL/PostgreSQL-compatible Aurora | Same engines |
| Exam cue | "business hours only", "unpredictable", "dev/test" | "steady", "24/7", "predictable" |

**Remember:** serverless removes the cost of idle hours but does not discount steady compute. Reserving an instance that sits idle makes you pay for the idle hours in advance.

### DynamoDB on-demand vs provisioned capacity

| | On-demand | Provisioned + auto scaling |
| --- | --- | --- |
| You pay for | Each read and write request | Capacity-unit-hours (RCUs/WCUs) |
| Planning | None | Set a floor and a ceiling |
| Cheapest for | Spiky, unknown, or new traffic | Steady or predictably cyclical traffic |
| Failure mode | Paying a per-request premium on steady volume | Over-provisioning for a past spike, or throttling when set too low |
| Exam cue | "unpredictable", "infrequent spikes" | "consistent and predictable" |

**Remember:** on-demand is not automatically the modern answer. For steady high volume it is the more expensive option.

### VPC peering vs Transit Gateway

| | VPC peering | Transit Gateway |
| --- | --- | --- |
| Topology | Point-to-point, non-transitive | Hub with transitive routing |
| Fees beyond data transfer | None | Per-attachment hourly + per GB processed |
| Connections for full mesh of N VPCs | N×(N−1)/2 | N attachments |
| Hybrid on-ramps | Not shared | Shares VPN and Direct Connect |
| Exam cue | "four VPCs, no growth expected" | "dozens of VPCs across accounts" |

**Remember:** for a small, stable number of VPCs, peering is cheaper. Transit Gateway's fees pay off once managing a mesh becomes the real problem.

[← Back to the study guide](README.md)
