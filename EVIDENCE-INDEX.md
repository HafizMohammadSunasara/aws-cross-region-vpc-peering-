# Evidence Index

This file explains what each screenshot proves.

| File | What it shows | Why it matters |
|---|---|---|
| `01-architecture-diagram.png` | Planned cross-region network design | Explains the intended architecture |
| `02-london-ec2-instance.png` | London EC2 instance details and private IP `172.31.23.144` | Confirms the London-side compute node exists |
| `03-virginia-ec2-instance.png` | Virginia EC2 instance details and private IP `10.0.137.157` | Confirms the Virginia-side compute node exists |
| `04-vpc-peering-connection-active.png` | VPC peering connection `London-to-Virginia` in `Active` state | Confirms the inter-VPC connection is established |
| `05-ping-virginia-to-london.png` | Successful ping from Virginia to London private IP | Proves one direction of private communication |
| `06-ping-london-to-virginia.png` | Successful ping from London to Virginia private IP | Proves the reverse direction of private communication |

## Raw vs Enhanced

- `evidence/raw/` contains the original screenshots.
- `evidence/enhanced/` contains enlarged, readability-improved versions for easier viewing.
