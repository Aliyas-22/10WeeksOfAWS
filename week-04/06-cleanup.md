# Week 4 Cleanup

Capture sanitized evidence first. Check `ap-south-1` and every other Region
used.

## Order

1. Unmount `/restore` and `/data`.
2. Terminate:
   - `cloudadhar-ec2-ami-builder-01`
   - `cloudadhar-ec2-ami-test-v2-01`
   - `cloudadhar-ec2-storage-lab-01`
3. Delete:
   - `cloudadhar-ebs-gp3-data-01`
   - `cloudadhar-ebs-gp3-restored-01`
   - Any orphaned Week 4 root or data volume
4. Deregister Golden AMI v1, if created, and v2.
5. Delete the AMIs' backing snapshots after deregistration.
6. Delete `cloudadhar-snap-gp3-data-01`.
7. Delete `cloudadhar-dlm-daily-ebs-snapshots`, if created.
8. Delete the three placement groups.
9. Delete `cloudadhar-sg-nginx-public` after dependencies are gone.
10. Delete the SSM role only if it was created exclusively for this lab.

## Final Check

- [ ] No Week 4 instance is running or stopped.
- [ ] No Week 4 EBS volume remains.
- [ ] No Week 4 AMI remains registered.
- [ ] No Week 4 snapshot remains.
- [ ] No DLM policy remains active.
- [ ] No Week 4 placement group remains.
- [ ] No unused security group, instance profile, or role remains.
- [ ] Every Region used was checked.

A stopped instance can still create costs through EBS, snapshots, AMIs, public
IPv4, and recurring lifecycle policies. Review billing later because usage data
can arrive after deletion.
