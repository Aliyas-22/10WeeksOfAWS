# Day 8 Lab - EBS Persistence and Snapshot Recovery

## Resources

| Resource | Exact name |
|---|---|
| Storage EC2 | `cloudadhar-ec2-storage-lab-01` |
| Data volume | `cloudadhar-ebs-gp3-data-01` |
| Snapshot | `cloudadhar-snap-gp3-data-01` |
| Restored volume | `cloudadhar-ebs-gp3-restored-01` |
| DLM policy | `cloudadhar-dlm-daily-ebs-snapshots` |
| Placement groups | `cloudadhar-pg-cluster-demo`, `cloudadhar-pg-spread-demo`, `cloudadhar-pg-partition-demo` |

Tag resources with `Project=AWS-Zero-to-Hero`, `Module=EC2 Storage`,
`Environment=Training`, `Owner=CloudAdhar`, `ManagedBy=Manual`,
`CleanupAfter=26 July 2026`, and `DataClassification=Training-Only`.

## Safety Warning

`mkfs` destroys the filesystem on the selected device. Run `lsblk -f`, identify
the new empty 2 GiB disk, and confirm it is not the root disk. Replace
`<new-device>` only with the verified path. Never format the restored volume.

## Build and Mount

1. Launch `cloudadhar-ec2-storage-lab-01` using Amazon Linux 2023, the SSM
   role, no public SSH, and IMDSv2 required.
2. Record its Availability Zone.
3. Create `cloudadhar-ebs-gp3-data-01` as 2 GiB gp3 in exactly the same AZ.
4. Attach the volume and connect through Session Manager.

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo blkid
df -hT
```

After verifying the new empty device:

```bash
sudo mkfs -t xfs <new-device>
sudo mkdir -p /data
sudo mount <new-device> /data
echo "Week 4 EBS test - $(date -u +%FT%TZ)" \
  | sudo tee /data/week4-proof.txt
sudo sync
sudo cat /data/week4-proof.txt
```

Get the UUID with `sudo blkid <new-device>`, back up `/etc/fstab`, and add:

```text
UUID=<filesystem-uuid> /data xfs defaults,nofail 0 2
```

Validate before reboot or stop/start:

```bash
sudo cp /etc/fstab /etc/fstab.week4-backup
sudo umount /data
sudo mount -a
findmnt /data
sudo cat /data/week4-proof.txt
```

Stop and start the instance, reconnect, and prove `/data` and the file persist.

## Snapshot and Restore

1. Run `sudo sync`.
2. Create `cloudadhar-snap-gp3-data-01`.
3. Wait for `Completed`.
4. Restore `cloudadhar-ebs-gp3-restored-01` in the instance AZ.
5. Attach it and run `lsblk -f`.
6. Do not run `mkfs`.

When the original and restored XFS filesystems are attached together, their
UUIDs match. Mount the clone temporarily with:

```bash
sudo mkdir -p /restore
sudo mount -o nouuid <restored-device> /restore
sudo cat /restore/week4-proof.txt
```

The recovered file must match the original. Do not add the cloned UUID to
`/etc/fstab` while the original is attached.

## Placement Groups

Create without launching extra fleets:

- `cloudadhar-pg-cluster-demo` - Cluster
- `cloudadhar-pg-spread-demo` - Spread
- `cloudadhar-pg-partition-demo` - Partition

Explain the workload signal for each, capture sanitized evidence, and delete
them during cleanup.

## Evidence

- EC2 and volume in the same AZ
- `lsblk -f` before and after attachment
- XFS mounted at `/data`
- UUID-based `/etc/fstab`
- File present after stop/start
- Snapshot `Completed`
- Restored file read without formatting
- Three placement strategies
