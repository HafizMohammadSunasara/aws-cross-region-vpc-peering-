# Implementation

## Project Goal

Create a working cross-region private connectivity path between:

- a **London EC2 instance** in `eu-west-2`
- a **Virginia EC2 instance** in `us-east-1`

---

## Step-by-Step Implementation

### 1. Create the Two VPC Environments

Two separate VPC environments were prepared in different regions:

- **London VPC** with CIDR `172.31.0.0/16`
- **Virginia VPC** with CIDR `10.0.0.0/16`

These CIDRs do not overlap, which is required for VPC peering.

---

### 2. Launch EC2 Instances

One EC2 instance was launched in each region.

#### London EC2
- Region: `eu-west-2`
- Private IP: `172.31.23.144`
- Public IP visible in screenshot evidence

#### Virginia EC2
- Region: `us-east-1`
- Private IP: `10.0.137.157`
- Public IP visible in screenshot evidence

---

### 3. Create the VPC Peering Connection

A peering connection named **London-to-Virginia** was created between the two VPCs.

Once accepted, the connection moved into the **Active** state.

This is an essential milestone because route tables depend on a valid peering connection target.

---

### 4. Update Route Tables

Routes were configured so each VPC knew how to reach the other network.

#### London route logic
- Destination: `10.0.0.0/16`
- Target: VPC peering connection

#### Virginia route logic
- Destination: `172.31.0.0/16`
- Target: VPC peering connection

Without these routes, the instances would remain isolated even if the peering connection existed.

---

### 5. Validate Security Controls

For connectivity testing, security controls must allow the traffic being used during validation.

In this project, successful ping output proves that the required path and permissions were available during testing.

---

### 6. Verify the Final Result

Private-IP communication was tested in both directions:

- **Virginia → London**
- **London → Virginia**

Both tests returned successful ping responses, confirming the cross-region private route was operational.
