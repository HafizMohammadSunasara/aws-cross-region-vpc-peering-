# AWS Cross-Region VPC Peering — Private Connectivity, Verified End to End

**A hands-on cloud networking case study: connecting EC2 workloads in London and N. Virginia over private IP addresses.**

**Amazon VPC · Amazon EC2 · Inter-Region VPC Peering · Route Tables · Linux · Network Validation**

> **The engineering challenge:** Two application environments run in different AWS Regions. They need to exchange traffic without sending their inter-VPC communication over public internet paths. How do we connect them, control the routes, and prove that packets actually reach their destination?

**Result:** I configured a cross-Region VPC peering lab between **Europe (London, `eu-west-2`)** and **US East (N. Virginia, `us-east-1`)**. Retained evidence shows an **Active** peering connection, running EC2 instances in both regions, and **successful ICMP ping responses in both directions over private IPs**.

| Project | Scope | Evidence-backed outcome |
| :-- | :-- | :-- |
| Regions | London ↔ N. Virginia | Two EC2 instances shown **Running** |
| Networking | Two non-overlapping VPC CIDRs | A peering connection shown **Active** |
| Verification | Private IPv4 from each instance | **Bidirectional ICMP replies captured** |
| Type | Independent hands-on AWS lab | **Not a production deployment or SLA claim** |

**Jump to:** [Deep Dive](DEEP-DIVE.md) · [Architecture](#architecture--how-the-network-works) · [Implementation](#implementation--from-two-isolated-vpcs-to-connectivity) · [Real proof](#proof-of-work--original-aws-and-terminal-evidence) · [Troubleshooting](#troubleshooting-mindset) · [Documentation](#explore-the-complete-engineering-notes)

---

## Why I built this

Getting an EC2 instance online is not the hard part of cloud networking. The hard part is making **separate networks communicate deliberately, predictably, and securely**.

Consider a service running in London and another service in N. Virginia. They live in different VPCs, with different address ranges and route tables. An instance in one VPC cannot automatically use the other's private addresses merely because both workloads are on AWS.

I built this lab to answer three practical questions:

1. **Design:** How should the two VPC address spaces be chosen so that they can be peered?
2. **Routing:** What needs to change after the peering request becomes active?
3. **Proof:** How can I demonstrate actual end-to-end private connectivity instead of just showing a green AWS status indicator?

**My biggest takeaway:** *“Peering is Active” is a control-plane milestone. Successful private-IP traffic in both directions is the meaningful data-plane test.*

## Architecture — how the network works

[![Cross-region VPC peering: London and N. Virginia](diagrams/cross-region-vpc-peering-architecture.png)](diagrams/cross-region-vpc-peering-architecture.png)

*Click the architecture to open the enlarged diagram.*

### The two sides of the connection

| Detail | London VPC | N. Virginia VPC |
| :-- | :-- | :-- |
| AWS Region | `eu-west-2` | `us-east-1` |
| VPC CIDR (architecture) | `172.31.0.0/16` | `10.0.0.0/16` |
| EC2 private IP (shown in screenshots) | `172.31.23.144` | `10.0.137.157` |
| Route required to peer | `10.0.0.0/16 → pcx-*` | `172.31.0.0/16 → pcx-*` |
| Validation | Ping Virginia's private IP | Ping London's private IP |

The routing relationship is conceptually straightforward:

```text
London EC2 (172.31.23.144)                     Virginia EC2 (10.0.137.157)
          │                                                  │
London VPC: 172.31.0.0/16                      Virginia VPC: 10.0.0.0/16
          │                                                  │
Route to 10.0.0.0/16                            Route to 172.31.0.0/16
          └─────── Inter-Region VPC Peering ────────────────┘
                     Private address traffic
```

**Why these decisions matter:**

- **Non-overlapping CIDRs:** AWS cannot establish peering for overlapping VPC address ranges. This is a network design decision, not something to fix after deployment by adding another route.
- **An active connection is not sufficient:** The appropriate subnet-associated route tables must direct peer-destination traffic to the peering connection.
- **Security controls are separate:** Instance and subnet policies still need to permit the intended traffic (ICMP in this validation). Peering itself does not bypass security groups or network ACLs.
- **Private is not the same as unreachable from the internet:** The EC2 screenshots also show public IPs used for instance administration. The **tested inter-VPC traffic** used private IPs; this lab does **not** establish that the instances had no public exposure.

AWS documents that inter-Region peering traffic stays on its private global backbone rather than traversing the public internet. See [AWS: How VPC peering works](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html).

## Implementation — from two isolated VPCs to connectivity

The project followed this engineering sequence:

| Stage | Work performed / design requirement | How it was checked |
| :-- | :-- | :-- |
| **01 — Network plan** | Identify two AWS Regions and non-overlapping VPC CIDRs | Architecture diagram and private IP details |
| **02 — Compute** | Prepare EC2 workloads in London and N. Virginia | AWS EC2 screenshots show both instances **Running** |
| **03 — Peering** | Request and accept inter-Region VPC peering | AWS VPC screenshot shows `London-to-Virginia` as **Active** |
| **04 — Routing and permissions** | Allow traffic toward the peer CIDR using peering routes and applicable security rules | Successful private-IP traffic confirms a usable path at test time; individual route/SG configurations are not separately captured |
| **05 — Test A** | Send ICMP from Virginia to `172.31.23.144` | Terminal replies captured |
| **06 — Test B** | Send ICMP from London to `10.0.137.157` | Terminal replies captured |

The recorded project is a **console-led AWS lab**. I have documented its implementation logic and observed outcomes. I have **not** added an invented Terraform stack or claimed a scripted build that the retained artifacts cannot substantiate.

**Want to understand every component from scratch?** Read the **[Engineering Deep Dive — beginner to advanced](DEEP-DIVE.md)**.

**For the fuller explanation:** [Implementation runbook](docs/IMPLEMENTATION.md) · [Network architecture decisions](docs/ARCHITECTURE.md).

## Proof of work — original AWS and terminal evidence

This repository preserves **both original screenshots and enlarged, lightly sharpened copies**. Enhanced images are easier to read; they are **not additional independent tests**.

| # | Actual artifact | What it establishes |
| :-- | :-- | :-- |
| 01 | [Architecture diagram](evidence/enhanced/01-architecture-diagram.png) | Designed London ↔ Virginia connectivity, VPC CIDRs and conceptual routes |
| 02 | [London EC2](evidence/enhanced/02-london-ec2-instance.png) | London-side EC2 is running; private IP `172.31.23.144` |
| 03 | [Virginia EC2](evidence/enhanced/03-virginia-ec2-instance.png) | Virginia-side EC2 is running; private IP `10.0.137.157` |
| 04 | [Active peering](evidence/enhanced/04-vpc-peering-connection-active.png) | `London-to-Virginia` connection has **Active** status |
| 05 | [Virginia → London ping](evidence/enhanced/05-ping-virginia-to-london.png) | Replies from London private IP `172.31.23.144` |
| 06 | [London → Virginia ping](evidence/enhanced/06-ping-london-to-virginia.png) | Replies from Virginia private IP `10.0.137.157` |

### Evidence highlight: connectivity in both directions

**Virginia → London — private-IP replies observed**

[![Virginia to London ping results](evidence/enhanced/05-ping-virginia-to-london.png)](evidence/enhanced/05-ping-virginia-to-london.png)

**London → Virginia — private-IP replies observed**

[![London to Virginia ping results](evidence/enhanced/06-ping-london-to-virginia.png)](evidence/enhanced/06-ping-london-to-virginia.png)

The visible terminal replies were in the neighborhood of **76 ms round-trip time** during the captured sessions. This is an observation from the screenshots, **not** a benchmark, latency guarantee or sustained availability measurement.

> **Evidence boundaries:** The peering-status screenshot also contains two failed connection entries; their exact failure reasons are not visible, so I do not attribute causes. The available screenshots do not separately verify route-table entries, every firewall rule, DNS resolution, throughput, encryption controls, application-layer traffic or a production security review. These are distinct next-step tests.

[View all screenshots and evidence descriptions →](docs/EVIDENCE-INDEX.md)

## Troubleshooting mindset

A working peering connection depends on several independent layers. If the next private-IP request fails, I would investigate in this order:

1. **Addressing:** Are the VPC CIDRs overlapping? Am I testing the correct private IP and Region?
2. **Peering state:** Is the connection really `Active`, and does it reference the intended requester and accepter VPCs?
3. **Route tables:** Does each **subnet-associated** route table have a route to the other CIDR via the correct peering ID? Is return traffic routed correctly?
4. **Access controls:** Do security groups, network ACLs and host firewalls permit the chosen protocol?
5. **Instance state:** Are both nodes healthy and listening/responding? Can I test from the actual source instance?
6. **Evidence:** What do AWS connection state, terminal results and packet tests prove—and what remains untested?

These are **diagnostic steps**, not claims that each failure occurred during this project. The screenshots show successful ICMP tests, but not a full root-cause history for the failed peering attempts visible in the console.

[Read troubleshooting scenarios →](docs/TROUBLESHOOTING.md)

## Design trade-offs and what I would improve next

**Why VPC peering?** For two VPCs, it provides a direct point-to-point relationship and makes it easy to reason about routes. However, peering is **not transitive**: adding VPC C does not let it communicate through VPC A to VPC B. At larger scale, I would compare it with AWS Transit Gateway or other hub-and-spoke options.

**Next experiments worth adding:**

- Export or capture the **actual route tables, security groups and NACL configurations** used during the test.
- Test a simple private TCP application between the two hosts, not only ICMP reachability.
- Review minimum-needed inbound permissions and restrict administrative access.
- Validate private DNS behavior where relevant.
- Record repeatable test commands, failure scenarios, cleanup actions and AWS data-transfer costs.
- Optionally rebuild the topology as Infrastructure as Code **after** validating against the original console setup.

The completed work demonstrates **cross-region network connectivity in a controlled lab**, not high availability, production readiness or measured disaster recovery.

## Explore the complete engineering notes

The **[DEEP-DIVE.md](DEEP-DIVE.md)** is a guided technical handbook—from simple networking analogies through packet forwarding, routes, ICMP, security groups and network ACLs, troubleshooting, DNS, cost, and interview preparation. It distinguishes captured outcomes from optional future experiments.

| File | Why a reviewer might open it |
| :-- | :-- |
| **[DEEP-DIVE.md](DEEP-DIVE.md)** | **Start-to-advanced tutorial, troubleshooting playbook, technical trade-offs, and interview prep** |
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | Address planning, components and traffic flow |
| [IMPLEMENTATION.md](docs/IMPLEMENTATION.md) | Sequence of VPC, EC2, peering and routing work |
| [VALIDATION.md](docs/VALIDATION.md) | What the running instances and bidirectional tests establish |
| [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | Common failure modes and diagnostic approach |
| [EVIDENCE-INDEX.md](docs/EVIDENCE-INDEX.md) | Screenshot-by-screenshot evidence explanation |
| [diagrams/](diagrams/) | Enlarged reference architecture |
| [evidence/raw/](evidence/). | My Original Screen Shots

### Repository layout

```text
README.md
├── DEEP-DIVE.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── IMPLEMENTATION.md
│   ├── VALIDATION.md
│   ├── TROUBLESHOOTING.md
│   └── EVIDENCE-INDEX.md
├── diagrams/
│   └── cross-region-vpc-peering-architecture.png
└── evidence/
    ├── raw/
    └── enhanced/
```

## About this project

**Project period:** July 2026 (hands-on implementation; evidence captured during the lab)  
**Author:** Hafiz Mohammad Sunasara — Cloud & DevOps Engineering Learner  
**Portfolio focus:** AWS networking, reliability, Linux, infrastructure design and hands-on troubleshooting.

[GitHub profile](https://github.com/HafizMohammadSunasara) · [LinkedIn](https://www.linkedin.com/in/hafiz-mohammad-sunasara-118756366/)

*This is a public engineering case study of a training project. Screenshots capture point-in-time lab state, not an ongoing live AWS environment. Before publishing evidence, review visible account identifiers, public IPs and any potentially sensitive account details.*
