# Week 4 Submission Format

```text
submissions/week-04/<github-username>/
├── README.md
├── architecture.png
└── evidence/
    ├── day7-ec2/
    ├── day8-storage/
    └── cleanup/
```

## README Template

```markdown
# Week 4 - EC2 Essentials, EBS, and Pricing

## Learner
- Name:
- GitHub:
- LinkedIn:
- Region:

## Day 7
- Instance selection:
- User Data result:
- IMDSv1 expected-deny:
- IMDSv2 result:
- Golden AMI validation:
- Pricing decisions:

## Day 8
- Instance and volume AZ:
- Filesystem and mount:
- Stop/start persistence:
- Snapshot recovery:
- Storage decisions:
- Placement decisions:

## Architecture Decision
Write 200-300 words.

## Cleanup
- Instances:
- Volumes:
- AMIs:
- Snapshots:
- DLM policies:
- Placement groups:
- Regions checked:

## Reflection
1. What EC2 decision mattered most?
2. What makes the formatting step dangerous?
3. What would you automate in production?
```

## Evidence Checklist

- [ ] Builder role and IMDSv2 required
- [ ] nginx and cloud-init success
- [ ] Tokenless `401` and token-based metadata success
- [ ] Golden AMI v2 available
- [ ] Test instance serves nginx without User Data
- [ ] Same-AZ EC2 and EBS
- [ ] `lsblk`, XFS, UUID mount, and persistence
- [ ] Completed snapshot and recovered file
- [ ] Placement-group strategies
- [ ] Architecture, cleanup, and LinkedIn link

Mask account IDs, ARNs, IPs, instance/volume/AMI/snapshot IDs, metadata tokens,
credentials, private keys, billing information, and organization URLs.
