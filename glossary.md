# SAA-C03 Glossary

167 SAA-C03 terms and services, each in one plain-English sentence.

[A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [R](#r) · [S](#s) · [T](#t) · [V](#v) · [W](#w)

## A

- **AbortIncompleteMultipartUpload**: A lifecycle action that deletes the parts of multipart uploads that were never completed.
- **Amazon API Gateway**: A managed front door for APIs that handles throttling, authorization, validation, and caching in front of a backend.
- **Amazon Data Lifecycle Manager**: A service that automates the creation and deletion of EBS snapshots on a schedule.
- **Amazon EventBridge**: An event bus whose rules match events by content and route them to targets such as Lambda, SQS, or Step Functions.
- **Amazon GuardDuty**: A threat detection service that analyzes CloudTrail events, VPC Flow Logs, and DNS logs.
- **Amazon Macie**: A service that discovers and classifies sensitive data such as PII in S3.
- **Amazon MSK**: A managed Apache Kafka service for applications built on Kafka APIs.
- **Amazon RDS Proxy**: A managed connection pool that lets a database be sized for its queries instead of its connection count.
- **Amazon SNS**: A publish/subscribe service that pushes a copy of each published message to every subscriber of a topic.
- **Amazon SQS**: A managed message queue that stores work durably until a consumer retrieves, processes, and deletes it.
- **Apache Parquet**: A compressed columnar file format that lets query engines read only the columns a query uses.
- **Aurora Capacity Unit (ACU)**: The unit of compute and memory that Aurora Serverless v2 scales and bills.
- **Aurora Global Database**: An Aurora configuration that replicates a cluster to other Regions for local reads and disaster recovery.
- **Aurora reader endpoint**: A single Aurora endpoint that spreads read connections across all of a cluster's replicas.
- **Aurora Serverless v2**: An Aurora option that adjusts database capacity automatically for variable workloads.
- **AWS Backup**: A service that centrally schedules, retains, and copies backups across AWS services, Regions, and accounts.
- **AWS Backup Vault Lock**: A feature that makes backups in an AWS Backup vault immutable.
- **AWS Batch**: A managed service that queues large numbers of batch jobs and provisions compute to run them.
- **AWS Certificate Manager (ACM)**: A service that issues and renews TLS certificates for integrated services such as ALB and CloudFront.
- **AWS CloudHSM**: A service that provides dedicated single-tenant hardware security modules in your VPC, under your exclusive control.
- **AWS Compute Optimizer**: A service that analyzes utilization history and recommends better-sized instances, Lambda memory, and EBS volumes.
- **AWS Control Tower**: A managed service that sets up a governed multi-account landing zone on top of Organizations.
- **AWS DataSync**: An online service that copies data into or between AWS storage services, once or on a schedule.
- **AWS Direct Connect**: A dedicated private physical connection into AWS that does not encrypt traffic by default.
- **AWS Fargate**: A serverless compute engine that runs containers without you managing any EC2 instances.
- **AWS IAM Identity Center**: The workforce single sign-on service that gives employees access to every account in an organization.
- **AWS Lake Formation**: A service that centrally manages database-, table-, and column-level permissions for an S3 data lake.
- **AWS Organizations**: The service that groups AWS accounts under a management account for central policy and consolidated billing.
- **AWS PrivateLink**: A way to expose one NLB-fronted service to other VPCs through interface endpoints without connecting the networks.
- **AWS Schema Conversion Tool (SCT)**: A tool that converts schemas and database code when migrating to a different engine.
- **AWS Secrets Manager**: A secrets store with managed automatic rotation and cross-account sharing.
- **AWS Shield**: A DDoS protection service for network- and transport-layer attacks, with a free Standard tier and a paid Advanced tier.
- **AWS Step Functions**: A service that runs multi-step workflows as state machines with branching, parallel steps, and per-step retries.
- **AWS STS**: The service that issues short-lived credentials when a principal assumes a role or federates in.
- **AWS Transfer Family**: A managed service that offers SFTP, FTPS, and FTP endpoints backed by S3 or EFS.
- **AWS Transit Gateway**: A regional hub that routes transitively between attached VPCs and on-premises connections, for attachment and processing fees.
- **AWS WAF**: A layer-7 firewall that filters HTTP(S) requests reaching CloudFront, ALB, API Gateway, or AppSync.
- **AWS X-Ray**: A tracing service that follows requests across distributed components and maps where latency and errors occur.

## B

- **Backlog per instance**: Visible SQS messages divided by running instances, published as a custom metric to scale worker fleets.
- **Backup and restore**: The DR strategy that stores only backups in the recovery Region and rebuilds the stack after a disaster.
- **Block storage**: Storage presented to the operating system as a raw disk that it formats and mounts, such as an EBS volume.
- **Blue-green deployment**: Running the new version beside the old one and shifting traffic over, which also allows an immediate rollback.
- **Burstable instance**: A T-family EC2 instance that runs at a baseline and bursts above it by spending CPU credits.
- **Byte-range fetch**: Downloading specific byte ranges of an S3 object, often in parallel.

## C

- **Cluster placement group**: A placement strategy that keeps instances physically near each other inside one AZ to minimize network latency.
- **Cognito identity pool**: A Cognito feature that exchanges identity tokens for temporary AWS credentials, including for guest users.
- **Cognito user pool**: A user directory that handles sign-up and sign-in for an application and issues JWTs.
- **Compute Savings Plan**: An hourly spend commitment whose discount applies across EC2 families, Regions, and operating systems, and also to Fargate and Lambda.
- **Confused deputy**: An attack in which a trusted service is induced to act on one customer's resources on behalf of another.
- **Convertible Reserved Instance**: A reservation that can be exchanged for another family, size, or OS during its term, at a smaller discount.
- **Cross-AZ data transfer**: Traffic between Availability Zones in a Region, charged per GB in each direction.
- **Cross-zone load balancing**: Letting a load balancer node send traffic to targets in every enabled Availability Zone.
- **Customer managed key**: A KMS key you create, whose key policy, rotation, and cross-account use you control.

## D

- **Dead-letter queue (DLQ)**: A separate queue that receives messages after they fail processing a set number of times.
- **Detective control**: A Control Tower rule, implemented with AWS Config, that flags noncompliant resources after the fact.
- **DynamoDB Accelerator (DAX)**: An API-compatible in-memory cache for DynamoDB that returns cached reads in microseconds.
- **DynamoDB on-demand mode**: A DynamoDB capacity mode that bills per request with no capacity planning.
- **DynamoDB provisioned mode**: A DynamoDB capacity mode that bills for configured read and write capacity units, usually paired with auto scaling.

## E

- **EBS Multi-Attach**: An io2 feature that lets several instances in one AZ share a volume, provided the software is cluster-aware.
- **EBS Snapshots Archive**: A lower-cost tier for rarely restored snapshots, with restores that take hours.
- **EC2 hibernation**: Stopping an instance after saving its RAM to the EBS root volume so it resumes with its memory intact.
- **EC2 Instance Savings Plan**: An hourly spend commitment tied to one instance family in one Region.
- **EFS lifecycle management**: An EFS policy that moves files that have not been accessed for a set period into a cheaper storage class.
- **EFS provisioned throughput**: An EFS setting that fixes throughput independently of how much data the file system holds.
- **Envelope encryption**: Encrypting data with a data key and then encrypting that data key with a KMS key.
- **Equal-cost multipath (ECMP)**: Routing across several VPN tunnels on a Transit Gateway to combine their bandwidth.
- **Explicit deny**: A policy statement that blocks an action and overrides any allow from any other policy.
- **Exponential backoff with jitter**: A retry approach that waits longer, with randomness added, after each throttled or failed call.
- **External ID**: A value required in a role's trust policy that prevents a third party from being tricked into using your role.

## F

- **Failover routing**: A Route 53 policy that sends all traffic to a primary record and switches to a secondary when the primary is unhealthy.
- **Fan-out**: A pattern that delivers one event to several independent consumers in parallel, usually through an SNS topic with one SQS queue per consumer.
- **Fargate launch type**: A way to run ECS or EKS tasks on AWS-managed capacity without managing EC2 instances.
- **Fault tolerance**: A design that keeps operating without any interruption when a component fails, because redundant capacity is already running.
- **File storage**: A shared directory hierarchy with locking and permissions that many clients reach over a network protocol.
- **FSx for Lustre**: A managed parallel file system for high-performance computing that can present an S3 bucket's objects as files.

## G

- **Gateway endpoint**: A free route-table target that gives private access to S3 or DynamoDB from inside a VPC.
- **Gateway Load Balancer**: A load balancer that places a fleet of third-party inspection appliances inline in the traffic path.
- **Gateway VPC endpoint**: A free route-table target that sends traffic to S3 or DynamoDB without passing through a NAT gateway.
- **Global Accelerator**: A service that gives an application two static anycast IPs and carries TCP or UDP traffic over the AWS backbone.
- **Global secondary index (GSI)**: A DynamoDB index with its own partition key that can be added to a table at any time.
- **Glue crawler**: A process that scans data, infers its schema and partitions, and records them in the Glue Data Catalog.
- **Glue Data Catalog**: A central metadata store of table definitions shared by Athena, EMR, and Redshift Spectrum.
- **gp3**: The general-purpose SSD EBS volume type, whose IOPS and throughput can be raised without enlarging the volume.
- **Graviton**: AWS-designed Arm processors that give better price-performance for compatible workloads.

## H

- **Heterogeneous migration**: A database migration between different engines, which needs SCT in addition to DMS.
- **High availability**: A design that recovers from component failures automatically and quickly, accepting a brief interruption.
- **Horizontal scaling**: Adding more instances behind a load balancer, instead of making one instance larger (vertical scaling).
- **Hot partition**: A DynamoDB partition that throttles because too many requests share the same partition key value.

## I

- **IAM role**: An identity with permissions but no permanent credentials, assumed to obtain temporary credentials.
- **Identity-based policy**: An IAM policy attached to a user, group, or role that grants that identity permissions.
- **Immutable infrastructure**: Replacing servers with newly built artifacts for every change instead of modifying them in place.
- **Instance profile**: The container that attaches an IAM role to an EC2 instance so its applications get temporary credentials.
- **Instance store**: Disk physically attached to the EC2 host that is very fast but loses its data when the instance stops or fails.
- **Instance warm-up**: The time an Auto Scaling group allows a new instance before counting its metrics toward scaling decisions.
- **Interface endpoint**: A PrivateLink network interface in your subnet that gives private access to most AWS services.
- **Interface VPC endpoint**: A PrivateLink network interface for private access to an AWS service, billed per hour per AZ and per GB.
- **io2**: The provisioned-IOPS SSD EBS volume type for sustained high IOPS with consistent low latency.
- **Isolated subnet**: A subnet with no route to an internet gateway or NAT gateway, so it has no internet path in either direction.

## K

- **Key policy**: The resource-based policy that is the final authority over who can use or manage a KMS key.
- **KMS custom key store**: A KMS feature that backs KMS keys with your own CloudHSM cluster.

## L

- **Latency-based routing**: A Route 53 routing policy that answers each query with the Region that has the lowest latency for the user.
- **Lazy loading**: A caching strategy that fills the cache only after a read misses it.
- **Least privilege**: Granting only the specific actions, resources, and conditions a principal actually needs.
- **Local secondary index (LSI)**: A DynamoDB index that keeps the table's partition key with a different sort key and is defined at table creation.
- **Loose coupling**: Connecting components through an intermediary so each depends only on that intermediary, not on the other component's availability or capacity.

## M

- **Minimum storage duration**: The number of days an S3 class bills an object for, even if the object is deleted or transitioned sooner.
- **Mixed instances policy**: An Auto Scaling group setting that combines an On-Demand base, a Spot share, and multiple instance types.
- **Multi-site active-active**: The DR strategy that serves users from two or more Regions simultaneously, each at production capacity.
- **Multipart upload**: Uploading a large S3 object as parts in parallel, with failed parts retried individually.

## N

- **NAT gateway**: A managed service in a public subnet that gives private-subnet instances outbound-only internet access.
- **NAT instance**: A self-managed EC2 instance that performs NAT, cheaper than a NAT gateway but limited by its own bandwidth and availability.
- **Network ACL**: A stateless subnet-level filter with numbered allow and deny rules.
- **Network Load Balancer**: A layer 4 load balancer for TCP and UDP that supports static IPs and very low latency.
- **Noncurrent version**: A previous copy of an object in a versioned bucket, billed at full price until a lifecycle rule removes it.

## O

- **Object storage**: Data kept as whole objects with keys and metadata and accessed through an HTTP API, as in Amazon S3.
- **Orchestration**: Coordinating a workflow from one central controller that tracks each step, in contrast to choreography, where services react to events independently.
- **Organizational unit (OU)**: A group of accounts within an organization where policies such as SCPs are attached and inherited.

## P

- **Partition placement group**: A placement strategy that splits instances into groups that do not share underlying hardware.
- **Partitioning**: Organizing data by key prefix so queries filtered on the partition column skip irrelevant data.
- **Permission set**: An Identity Center collection of policies that becomes a role in each account it is assigned to.
- **Permissions boundary**: A managed policy that caps the maximum permissions a user or role can have without granting any.
- **Pilot light**: The DR strategy that keeps a live, replicated data layer in the recovery Region with compute off until failover.
- **Predictive scaling**: An Auto Scaling policy that forecasts recurring load from history and adds capacity before the load arrives.
- **Preventive control**: A Control Tower rule, implemented as an SCP, that blocks a noncompliant action.
- **Price-capacity-optimized allocation**: A Spot allocation strategy that favors pools that are both cheap and unlikely to be interrupted.
- **Provisioned concurrency**: A Lambda setting that keeps execution environments initialized to avoid cold-start delays.

## R

- **RDS Multi-AZ**: An RDS deployment that keeps a synchronously replicated standby in another AZ and fails over to it automatically.
- **RDS Proxy**: A managed connection pool for RDS and Aurora that shortens effective failover time and protects the database from connection exhaustion.
- **Read replica**: An asynchronously updated, readable copy of a database, used to scale reads and able to be promoted to a standalone primary.
- **Recovery point objective (RPO)**: The largest acceptable data loss, measured as the age of the latest recoverable copy of the data.
- **Recovery time objective (RTO)**: The maximum downtime the business accepts before service must be back.
- **Redshift Spectrum**: A Redshift feature that queries data in S3 through the Glue Data Catalog alongside warehouse tables.
- **Requester Pays**: A bucket setting that charges the downloader, not the owner, for request and data transfer costs.
- **Resource-based policy**: A policy attached to a resource, such as a bucket or KMS key, that names which principals may use it.
- **Retrieval fee**: A per-GB charge for reading data from S3 Infrequent Access and Glacier classes.
- **Route 53 health check**: A probe or alarm-based check that Route 53 uses to stop returning DNS records for unhealthy endpoints.

## S

- **S3 Bucket Key**: A bucket-level data key that reduces the number of KMS requests made under SSE-KMS.
- **S3 Cross-Region Replication (CRR)**: Asynchronous copying of new S3 objects to a bucket in a different Region.
- **S3 Glacier Deep Archive**: The lowest-cost S3 storage class, with restores of about 12 hours or more and a 180-day minimum.
- **S3 Intelligent-Tiering**: An S3 class that moves each object between access tiers based on its own usage, for a monitoring fee and with no retrieval fees.
- **S3 Lifecycle configuration**: Bucket rules that transition objects to colder classes or delete them at a set age.
- **S3 Object Lock**: WORM protection that prevents object versions from being deleted or overwritten during a retention period.
- **S3 One Zone-IA**: A lower-cost infrequent-access class that keeps data in a single Availability Zone, so it fits only re-creatable or duplicated data.
- **S3 Transfer Acceleration**: Routing uploads through the nearest edge location and the AWS backbone to speed long-distance transfers.
- **S3 Versioning**: An S3 feature that keeps every version of an object so accidental deletions and overwrites can be undone.
- **SecureString parameter**: A KMS-encrypted value in Systems Manager Parameter Store, suitable for secrets that do not need managed rotation.
- **Security group**: A stateful, allow-only firewall attached to a resource's network interface.
- **Service control policy (SCP)**: An Organizations guardrail that limits the maximum permissions in member accounts and never grants access.
- **Shard**: A unit of capacity in a Kinesis data stream that keeps records with the same partition key in order.
- **Single point of failure (SPOF)**: Any component whose failure alone takes the whole workload down.
- **Spot capacity pool**: The spare capacity for one instance type in one Availability Zone, which AWS can reclaim independently of other pools.
- **SQS FIFO queue**: A queue type that preserves order within a message group and processes each message exactly once, at a throughput cap.
- **SSE-KMS**: S3 server-side encryption with a KMS key, where each key use is logged in CloudTrail.
- **SSM Session Manager**: A Systems Manager feature that opens logged shell sessions to instances with no inbound ports or SSH keys.
- **Stateless design**: Keeping sessions, files, and pending work outside the instance so that any instance can handle any request.
- **Step scaling**: An Auto Scaling policy that applies different capacity changes at different CloudWatch alarm thresholds.
- **Storage Gateway**: An on-premises appliance that gives local applications file, volume, or tape access to data stored in AWS, with a local cache.
- **Subscription filter policy**: An SNS setting that limits the messages delivered to one subscriber to those matching given attributes.

## T

- **Target tracking policy**: An Auto Scaling policy that adjusts capacity to hold a chosen metric at a target value.
- **Target tracking scaling**: An Auto Scaling policy that adds or removes capacity to hold a chosen metric at a target value.
- **Tight coupling**: A design in which one component calls another synchronously and fails or slows whenever that component does.
- **Transit Gateway**: A regional hub that connects VPCs, VPNs, and Direct Connect with transitive routing.
- **Trust policy**: The part of a role that defines which principals are allowed to assume it.

## V

- **Visibility timeout**: How long a received SQS message stays hidden from other consumers before it reappears for another attempt.

## W

- **Warm standby**: The DR strategy that keeps a smaller but fully working copy of the workload running in the recovery Region.
- **Write sharding**: Adding a suffix to a DynamoDB partition key value so writes spread across several partitions.
- **Write-through**: A caching strategy that updates the cache on every database write.

[← Back to the study guide](README.md)
