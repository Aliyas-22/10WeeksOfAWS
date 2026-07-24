# Week 3 - Amazon VPC

## Name
Shaikh Aliya 

## Architecture

Built a **two-AZ VPC** (`cloudadhar-day6-vpc`) with a public + private tier in each zone.

**[Insert your architecture diagram screenshot here]**

Diagram includes:
- VPC boundary + both Availability Zones (ap-south-1a, ap-south-1b)
- 4 subnets with CIDR ranges
- Internet Gateway attached to VPC
- NAT Gateway inside public subnet
- Route tables + their routes
- Security Groups on each instance/ENI
- Custom NACL on public subnet
- S3 Gateway Endpoint + Interface Endpoint
- Flow Logs path to CloudWatch

## CIDR Plan

| Resource | CIDR | Total Addresses | Usable Addresses* |
|---|---|---|---|
| VPC | 10.10.0.0/16 | 65,536 | — |
| Public-A | 10.10.1.0/24 | 256 | 251 |
| Private-A | 10.10.11.0/24 | 256 | 251 |
| Public-B | 10.10.2.0/24 | 256 | 251 |
| Private-B | 10.10.12.0/24 | 256 | 251 |

*AWS reserves 5 IPs per subnet (network address, router, DNS, future use, broadcast).

 All subnets fit inside VPC range, no overlaps.

## Public vs Private

A subnet is public when...

- Its route table sends `0.0.0.0/0` **directly to an Internet Gateway**
- That's the *only* rule — not the name, not the auto-assign IP setting
- Private subnet = either no `0.0.0.0/0` route, or routes it to a **NAT Gateway** instead of an IGW

## Day 6 Result

**NAT Gateway + Private Egress**
- Created NAT Gateway `cloudadhar-day6-nat-a` in public subnet, with its own Elastic IP
- Dedicated private route table → `0.0.0.0/0 → NAT Gateway`
- Launched private EC2 instance (no public IP), connected via **Session Manager** (no SSH key/bastion needed)
- Ran `curl` to a public HTTPS endpoint → confirmed outbound internet works, with zero inbound exposure

**[Insert screenshot: NAT Gateway details]**

**Security Group vs NACL**
- Launched public EC2 with nginx → confirmed page loads over public IP
- Created custom NACL on public subnet, allow rules for HTTP + ephemeral return ports
- Added one **lower-numbered deny rule** blocking HTTP from my own IP (`/32`)
  - NACLs evaluate rules in order, stop at first match → deny hit before allow
-  Confirmed page failed to load → removed deny rule → access restored

**Key takeaway:**

| | Security Group | NACL |
|---|---|---|
| State | Stateful (return traffic auto-allowed) | Stateless (return traffic needs its own rule) |
| Rules | Allow only | Allow + Deny |
| Evaluation | All rules checked | Stops at first matching rule (in order) |

**[Insert screenshot: Route tables / NACL rules]**

**VPC Endpoints**
- **S3 Gateway Endpoint** → attached to private route table, adds private prefix-list route to S3, no hourly cost
  - Verified with an S3 API call from the private instance
- **Interface Endpoint** (EC2 API) → private DNS enabled, own restrictive SG
  - Verified via `nslookup` resolving to a private IP
  - Deleted right after testing (bills hourly)

**[Insert screenshot: Endpoints]**

**Flow Logs**
- Enabled VPC Flow Logs (traffic type: All) → CloudWatch Logs group
- Generated both normal traffic (web page load) and rejected traffic (re-triggered NACL deny)
- Queried CloudWatch Logs Insights → confirmed both **ACCEPT** and **REJECT** records, with source/destination, ports, protocol, action

**[Insert screenshot: CloudWatch Logs Insights results]**

## Architecture Decisions

**Why NAT per AZ?**
- Avoids cross-AZ dependency
- If AZ-A's private subnet routed through a NAT Gateway sitting in AZ-B, an AZ-B outage would break AZ-A's internet access too
- Keeping NAT local to each AZ = each AZ stays independently healthy — the whole point of building multi-AZ

**Why an S3 Gateway Endpoint?**
- Without it: private → S3 traffic goes through NAT → costs per-GB + extra hop
- With it: direct private route via managed prefix list
- No hourly charge, no NAT data-processing cost
- Preferred path specifically for **S3 and DynamoDB**

**When Transit Gateway instead of Peering?**
- VPC Peering = one-to-one only, and **non-transitive**
  - A↔B and B↔C peered ≠ A can reach C
- Gets unmanageable past a handful of VPCs (needs a separate peering link per pair)
- **Transit Gateway** = central transitive hub
  - Each VPC connects once → can reach every other connected VPC through it

## Where I Got Stuck

- **Problem:** Private EC2's SSM agent went from Online → Offline mid-testing, reporting a connection timeout to the SSM service
- **Investigated, layer by layer:**
  - ✅ Private subnet route table → correct (NAT target)
  - ✅ Public subnet route table → correct (IGW target)
  - ✅ NAT Gateway status + Elastic IP → healthy
  - ✅ Security group outbound rules → fully open
  - ✅ Custom NACL (public subnet) → correct, not blocking
  - ✅ Default NACL (private subnet) → fine
  - ✅ IGW attachment → confirmed attached
- **Result:** every individual layer looked correct on its own — issue was in how several recent changes (route tables, SGs, endpoint creation) interacted together, not one single broken setting
- **Fix:** rebooted the instance to clear stale network state; also Removed the subnet Associate from NACl

## Cleanup

- ✅ Interface Endpoint — deleted right after testing (hourly billed)
- ✅ Public + private EC2 test instances — terminated
- ✅ NAT Gateway — deleted
- ✅ Elastic IP (NAT's) — released
- ✅ Custom NACL — disassociated, then deleted
- ✅ Extra private route table — disassociated, deleted
- ✅ Public + private route tables — deleted (main RT kept)
- ✅ Internet Gateway — detached, deleted
- ✅ All 4 subnets — deleted
- ✅ VPC — deleted
- ✅ S3 Gateway Endpoint — deleted (optional, no hourly cost)


## LinkedIn Posts

- Day 5: [Insert your Day 5 LinkedIn post URL]
- Day 6: [Insert your Day 6 LinkedIn post URL]
