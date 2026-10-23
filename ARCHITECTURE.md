# Architecture

## Objective

Build a private network path between two AWS environments in different regions:

- **London (eu-west-2)**
- **N. Virginia (us-east-1)**

The design goal is to allow two EC2 instances to communicate over their **private IPs**, not through public internet addressing.

---

## Core Components

| Component | London Side | Virginia Side |
|---|---|---|
| Region | `eu-west-2` | `us-east-1` |
| VPC CIDR | `172.31.0.0/16` | `10.0.0.0/16` |
| Compute | 1 EC2 instance | 1 EC2 instance |
| Connectivity | VPC Peering | VPC Peering |
| Routing | Route to `10.0.0.0/16` | Route to `172.31.0.0/16` |

---

## Architecture Logic

A VPC peering connection creates a direct private networking relationship between two VPCs. However, the peering connection alone is **not enough**.

To make the setup functional, these elements must work together:

1. Non-overlapping CIDR ranges
2. Active VPC peering connection
3. Route table entries pointing to the peering connection
4. Security groups that allow the required validation traffic
5. Healthy EC2 instances in both VPCs

---

## Traffic Flow

### London → Virginia

- Source instance private IP: `172.31.23.144`
- Destination instance private IP: `10.0.137.157`
- London route table forwards `10.0.0.0/16` traffic to the peering connection

### Virginia → London

- Source instance private IP: `10.0.137.157`
- Destination instance private IP: `172.31.23.144`
- Virginia route table forwards `172.31.0.0/16` traffic to the peering connection

---

## Why VPC Peering?

This design is useful when:

- two environments exist in different regions,
- workloads need private communication,
- and the setup should remain relatively simple without introducing a transit architecture.

---

## Limitation to Remember

VPC Peering does **not** provide transitive routing.
If a third VPC is added later, peering does not automatically allow that third network to communicate through an already-peered VPC.
