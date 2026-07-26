# Week 4 Quick Revision

## Recall

1. AMIs are Regional; EBS volumes are Availability Zone scoped.
2. Golden AMIs are approved, patched, hardened, tested, and versioned.
3. User Data normally runs at first boot through cloud-init.
4. IMDSv2 requires a token; a tokenless request should return `401`.
5. Reserved Instances and Savings Plans are discounts, not server objects.
6. gp3 fits general workloads; io2 Block Express fits critical provisioned
   IOPS.
7. st1 and sc1 are throughput HDD options and cannot be boot volumes.
8. Snapshots are incremental Regional recovery points.
9. Instance Store is for temporary reproducible data.
10. Cluster optimizes proximity, Spread isolates instances, and Partition
    separates distributed-system failure domains.

## Scenarios

| Requirement | Best choice |
|---|---|
| Unknown short-term demand | On-Demand |
| Interruptible checkpointed batch | Spot |
| Steady flexible compute spend | Savings Plans |
| Server-bound licensing | Dedicated Host |
| Repeatable patched fleet | Golden AMI and Launch Template |
| General-purpose boot/data volume | gp3 |
| Sustained critical IOPS | io2 Block Express |
| Immediate restored-volume performance | Fast Snapshot Restore |
| Automatic snapshot retention | DLM |
| Temporary host-local cache | Instance Store |
| Tightly coupled HPC | Cluster placement group |
| Small critical isolated set | Spread placement group |
| Kafka rack-style separation | Partition placement group |

## Important Traps

- A lower-cost answer is wrong if it violates security, availability, RTO,
  latency, durability, or throughput requirements.
- An EBS volume cannot attach across AZs.
- Multi-Attach does not provide cross-AZ shared storage or write coordination.
- Do not run `mkfs` on a restored volume.
- A cloned XFS filesystem can require `mount -o nouuid` when mounted beside the
  original.
- A stopped instance can still leave billable storage, snapshots, AMIs, or
  public IPv4 resources.

## Exam Method

1. Identify hard requirements.
2. Identify failure scope and interruption tolerance.
3. Select compute, image, storage, or placement pattern.
4. Validate security, recovery, and operations.
5. Compare cost among choices that meet the requirements.

## Final Check

- [ ] I can choose an EC2 family and purchase option.
- [ ] I can explain Golden AMIs, User Data, and IMDSv2.
- [ ] I can select an EBS type from IOPS and throughput.
- [ ] I can safely mount and restore an EBS filesystem.
- [ ] I can choose a placement strategy.
- [ ] I know every resource that must be cleaned up.
