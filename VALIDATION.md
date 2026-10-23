# Validation

## Validation Objective

Confirm that the two EC2 instances can communicate across regions using **private IP addresses**.

---

## Evidence-Based Validation

### 1. Instance Availability

- London EC2 instance is shown in a **Running** state
- Virginia EC2 instance is shown in a **Running** state

Files:
- `evidence/enhanced/02-london-ec2-instance.png`
- `evidence/enhanced/03-virginia-ec2-instance.png`

---

### 2. Peering Connection Status

The VPC peering connection **London-to-Virginia** is shown as **Active**.

File:
- `evidence/enhanced/04-vpc-peering-connection-active.png`

---

### 3. Bidirectional Connectivity Tests

#### Test A — Virginia to London

From the Virginia instance, ping requests to `172.31.23.144` were successful.

File:
- `evidence/enhanced/05-ping-virginia-to-london.png`

#### Test B — London to Virginia

From the London instance, ping requests to `10.0.137.157` were successful.

File:
- `evidence/enhanced/06-ping-london-to-virginia.png`

---

## Final Validation Statement

The project goal was achieved because:

- both EC2 instances are running,
- the VPC peering connection is active,
- and both instances can reach each other over private IP space.

That combination confirms that the cross-region peering setup is functional.
