# Resource inventory

Region: Asia Pacific (Mumbai) / ap-south-1, Availability Zone ap-south-1a. All IDs are taken from my own console screenshots. Every resource was deleted after the lab (see `cleanup-and-learnings.md`).

## Network

| Resource | Name | ID | CIDR / IP | Evidence |
|---|---|---|---|---|
| VPC A | ewb-vpc-a | vpc-0fcced593290ff166 | 10.10.0.0/16 | screenshots/01-vpc-a.png |
| Public subnet | ewb-a-public | subnet-0242e6e0df78aaa9e | 10.10.1.0/24 | screenshots/02a-subnets-a-public.png |
| Private subnet | ewb-a-private | subnet-023b20eb02c8137da | 10.10.2.0/24 | screenshots/02b-subnets-a-private.png |
| Internet gateway | ewb-igw-a | igw-088353fc4cb5b4bff | attached to ewb-vpc-a | screenshots/03-igw-attached.png |
| Route table | ewb-rt-a-public | rtb-049e653e2a995ca2b | associated with ewb-a-public | screenshots/04a-rt-a-public.png |
| Route table | ewb-rt-a-private | rtb-0e91f5a2ab1f3bbc6 | associated with ewb-a-private | screenshots/21-rt-a-public-final.png |
| VPC B | ewb-vpc-b | vpc-032ad1b0062f2c9ce | 10.20.0.0/16 | screenshots/06-vpc-b.png |
| Private subnet | ewb-b-private | subnet-0399e45a6a15ea5a2 | 10.20.1.0/24 | screenshots/07-subnet-b.png |
| Route table | ewb-rt-b-private | rtb-0d898c941c3ab77c3 | associated with ewb-b-private | screenshots/08-rt-b-before-peering.png |
| Peering connection | ewb-a-b-peer | pcx-00d385cdc737a48a7 | requester VPC A, accepter VPC B | screenshots/20-peering-active.png |

VPC B has no internet gateway.

## Security groups

| Name | ID | VPC |
|---|---|---|
| ewb-sg-bastion | sg-04267939824583309 | ewb-vpc-a |
| ewb-sg-app-a | sg-0d9ac0dd174c7a0d8 | ewb-vpc-a |
| ewb-sg-app-b | sg-094f2003708f70c5f | ewb-vpc-b |

## Compute

All three instances are Amazon Linux 2023, t3.micro, in ap-south-1a.

| Name | Instance ID | Subnet | Private IP | Public IPv4 |
|---|---|---|---|---|
| ewb-bastion | i-0cd27abfec699fa83 | ewb-a-public | 10.10.1.108 | yes (changed during the lab, see note) |
| ewb-app-a | i-0c9ab388f415eca9e | ewb-a-private | 10.10.2.105 | none |
| ewb-app-b | i-057a7ebb8e0ae2bba | ewb-b-private | 10.20.1.108 | none |

Key pair: `ewb-lab` (RSA), key-03729087bf4dcbed9.

## Notes

- I used AWS-assigned private IPs instead of the workbook's example `.10` addresses. The workbook allows this if every instance IP and /32 rule uses the actual values, so all security group rules and tests in this repo use 10.10.1.108, 10.10.2.105 and 10.20.1.108.
- The bastion's public IPv4 was 3.108.67.58 in `screenshots/16-ssh-bastion.png` and 13.126.9.6 in a later session (`terminal-output/02-forward-a-to-b.txt`). It changed between sessions. A public IPv4 that is not an Elastic IP is released when an instance is stopped, so a new one is assigned on the next start.
- Tags used: `Project = EWB-VPC-Peering`, `Student = Aditya Kumar Gupta`.
