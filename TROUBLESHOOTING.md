# Troubleshooting Notes

## Common Problems in Cross-Region VPC Peering

### 1. Overlapping CIDR Ranges
**Problem:** Peering cannot work correctly if both VPCs use overlapping IP ranges.  
**Fix:** Use distinct CIDRs such as `172.31.0.0/16` and `10.0.0.0/16`.

---

### 2. Peering Connection Not Active
**Problem:** The peering request exists but is still pending or failed.  
**Fix:** Accept the peering request in the accepter region/account and verify the status becomes **Active**.

---

### 3. Missing Route Table Entries
**Problem:** The peering exists but traffic still cannot cross regions.  
**Fix:** Add a route in each VPC route table pointing the remote CIDR to the peering connection.

---

### 4. Security Group Blocks ICMP or Other Test Traffic
**Problem:** Instances are running but ping fails.  
**Fix:** Confirm the relevant inbound and outbound rules allow the test traffic during validation.

---

### 5. Testing the Wrong IP Address
**Problem:** A public IP is tested instead of the private IP.  
**Fix:** Use the instance's **private IP** for true VPC-peering validation.

---

### 6. Instance Reachability Issue
**Problem:** Network configuration is correct, but the instance itself is not healthy.  
**Fix:** Confirm EC2 status checks pass and the instance is accessible.

---

## Key Lesson

In AWS networking, an **Active peering connection alone is not proof of success**.

Real success is only confirmed after:

- routing is correct,
- security rules allow the traffic,
- and end-to-end validation works from one instance to the other.
