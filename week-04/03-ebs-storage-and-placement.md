# EBS Storage and Placement

Choose storage using IOPS, throughput, durability, persistence, access pattern,
failure scope, and cost.

## EBS Volume Types

| Type | Media | Best use |
|---|---|---|
| gp3 | SSD | General purpose, boot, applications |
| io2 Block Express | Provisioned IOPS SSD | Critical low-latency databases |
| st1 | Throughput HDD | Frequently accessed large sequential data |
| sc1 | Cold HDD | Infrequently accessed sequential data |

`st1` and `sc1` cannot be boot volumes. EBS volumes are Availability Zone
scoped and normally attach only to instances in the same AZ.

## Snapshots and Recovery

EBS snapshots are incremental point-in-time backups managed Regionally. They
can restore volumes into AZs in their Region and can be copied to another
Region.

- Quiesce applications when consistency matters.
- Restored volumes normally initialize blocks as they are read.
- Fast Snapshot Restore creates fully initialized volumes in selected AZs and
  adds cost.
- Data Lifecycle Manager automates tag-based snapshot and EBS-backed AMI
  creation and retention.
- Plan KMS permissions before copying or sharing encrypted snapshots.

## Multi-Attach and Instance Store

Multi-Attach supports compatible `io1` or `io2` volumes attached to multiple
Nitro instances in one AZ. The application and filesystem must coordinate
concurrent writes. It is not cross-AZ shared storage.

Instance Store is host-attached ephemeral storage. Use it for cache, buffer,
scratch, and replicated or reproducible data. Do not keep the only copy of
important data there. Stop, hibernate, terminate, or host failure can remove
the data.

## Placement Groups

| Strategy | Priority | Typical workload |
|---|---|---|
| Cluster | Low latency and high throughput | HPC and tightly coupled nodes |
| Spread | Distinct underlying hardware | Small set of critical instances |
| Partition | Failure-domain separation | Kafka, Hadoop, Cassandra |

Placement groups do not replace Multi-AZ design, replication, backups, or Auto
Scaling.

## Exam Cues

- General boot/application volume -> gp3
- Critical sustained provisioned IOPS -> io2 Block Express
- Frequent large sequential data -> st1
- Cold sequential data -> sc1
- Temporary reproducible local data -> Instance Store
- Immediate restored-volume performance -> Fast Snapshot Restore
- Automated snapshot retention -> DLM
- Tightly coupled compute -> Cluster
- Individual hardware isolation -> Spread
- Rack-aware distributed system -> Partition

## Official References

- [EBS volume types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- [EBS snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)
- [Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html)
- [Fast Snapshot Restore](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-fast-snapshot-restore.html)
- [Instance Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html)
- [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
