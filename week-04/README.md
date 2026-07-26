# Week 4 - EC2 Essentials, EBS, and Pricing Models

AWS Zero To Hero - CloudAdhar x TrainWithShubham<br>
Sessions: Jul 25-26, 2026<br>
Course sessions: Day 7-8<br>
Exam focus: SAA-C03 Domains 1-4<br>
Main pillars: Security, Reliability, Performance Efficiency, and Cost Optimization

Week 4 covers EC2 instance selection, secure bootstrapping, Golden AMIs,
IMDSv2, pricing models, EBS storage, snapshots, Instance Store, and placement
groups. Use the Day 7 and Day 8 Student Guides for detailed classroom support.

## Start Here

| Seq | Session | Focus | File |
|---:|---|---|---|
| 01 | Day 7 | EC2, AMIs, User Data, IMDSv2, and pricing | [01-ec2-fundamentals-and-pricing.md](./01-ec2-fundamentals-and-pricing.md) |
| 02 | Day 7 | Build, automate, and validate a Golden AMI | [02-ec2-ami-imdsv2-lab.md](./02-ec2-ami-imdsv2-lab.md) |
| 03 | Day 8 | EBS, snapshots, Instance Store, and placement | [03-ebs-storage-and-placement.md](./03-ebs-storage-and-placement.md) |
| 04 | Day 8 | Mount, snapshot, and restore gp3 storage | [04-ebs-snapshot-lab.md](./04-ebs-snapshot-lab.md) |
| 05 | Both | Document the architecture and decisions | [05-architecture-exercise.md](./05-architecture-exercise.md) |
| 06 | End | Remove all billable resources | [06-cleanup.md](./06-cleanup.md) |
| 07 | End | Submit Week 4 evidence | [07-submission-format.md](./07-submission-format.md) |
| 08 | Daily | Share learning progress | [08-linkedin-post.md](./08-linkedin-post.md) |
| 09 | Review | Revise the key decisions | [09-quick-revision.md](./09-quick-revision.md) |

## Required Outcomes

- Select EC2 families from compute, memory, storage, network, and accelerator
  requirements.
- Compare AWS, Marketplace, custom, and Golden AMIs.
- Explain mutable patching, immutable image replacement, and Image Builder.
- Bootstrap nginx with User Data.
- Require IMDSv2, prove tokenless access returns `401`, and use a token safely.
- Select On-Demand, Reserved Instances, Savings Plans, Spot, and Dedicated
  options from workload signals.
- Compare gp3, io2 Block Express, st1, and sc1.
- Attach, format, and persistently mount a gp3 volume by UUID.
- Create a snapshot and restore its data without formatting the restored
  volume.
- Explain Multi-Attach, Fast Snapshot Restore, DLM, and Instance Store.
- Select Cluster, Spread, and Partition placement groups.

## Lab Architecture

```text
Day 7
User Data + IMDSv2
        |
        v
cloudadhar-ec2-ami-builder-01
        |
        v
cloudadhar-ami-nginx-golden-v2-20260725
        |
        v
cloudadhar-ec2-ami-test-v2-01
        `-- nginx works without User Data

Day 8
cloudadhar-ec2-storage-lab-01
        |
        v
cloudadhar-ebs-gp3-data-01
        |
        v
cloudadhar-snap-gp3-data-01
        |
        v
cloudadhar-ebs-gp3-restored-01
```

Use `ap-south-1` unless instructed otherwise. An EBS volume and its target EC2
instance must be in the same Availability Zone.

## Minimum Submission

- Instance-family, storage, placement, and pricing decision tables
- Builder configuration showing the SSM role and IMDSv2 required
- nginx and cloud-init success
- IMDSv1 `401` and successful IMDSv2 token test
- Golden AMI v2 and test launch without User Data
- Same-AZ EBS attachment and `lsblk` evidence
- UUID-based `/etc/fstab` and stop/start persistence evidence
- Completed snapshot and recovered test file
- Architecture diagram, cleanup proof, and public learning post

## Cost and Safety

- Prefer Session Manager; never open SSH to `0.0.0.0/0`.
- Never store credentials in User Data, an AMI, or screenshots.
- Verify the empty block device before running `mkfs`.
- Never run `mkfs` on a volume restored from a snapshot.
- Stopped instances can still incur EBS, snapshot, AMI, and public IPv4 costs.
- Remove recurring DLM policies and check every Region used.
- Mask account IDs, ARNs, resource IDs, IPs, tokens, and billing data.

<div align="center">

[Week 3](../week-03/) | [Home](../README.md)

</div>
