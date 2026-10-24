# Security groups

Default outbound allow-all rules were kept. Security groups are stateful, so replies to allowed traffic are allowed automatically. Network ACLs were left as the default allow-all.

| Security group | VPC | Type / port | Source |
|---|---|---|---|
| ewb-sg-bastion (sg-04267939824583309) | ewb-vpc-a | SSH / TCP 22 | my public IPv4 /32 |
| ewb-sg-app-a (sg-0d9ac0dd174c7a0d8) | ewb-vpc-a | SSH / TCP 22 | 10.10.1.108/32 (bastion private IP) |
| ewb-sg-app-a | ewb-vpc-a | Custom ICMP IPv4, Echo Request | 10.20.1.108/32 (App-B) |
| ewb-sg-app-b (sg-094f2003708f70c5f) | ewb-vpc-b | SSH / TCP 22 | 10.10.1.108/32 (bastion private IP) |
| ewb-sg-app-b | ewb-vpc-b | Custom ICMP IPv4, Echo Request | 10.10.2.105/32 (App-A) |

## Evidence

- `screenshots/09-sg-bastion.png`: bastion group with its single SSH inbound rule.
- `screenshots/10-sg-app-a.png`: App-A group with SSH and Echo Request rules.
- `screenshots/11-sg-app-b.png`: App-B group with SSH and Echo Request rules.
- `screenshots/27-failure-rule-removed.png` and `screenshots/29-rule-restored.png` show the App-B rules with their sources (10.10.1.108/32 for SSH, 10.10.2.105/32 for ICMP).

The source column is cut off in screenshots 09, 10 and 11 because of the window width, so for the bastion and App-A groups the sources are shown in my architecture diagram (`diagrams/architecture.png`) and are also proven by behaviour: SSH through the bastion to App-A worked, and App-B to App-A ping worked. The App-B sources are directly visible in screenshots 27 and 29.

## Why /32 sources

A /32 allows exactly one host. SSH to the private servers is allowed only from the bastion, and ping is allowed only from the one peer host that needs it, so nothing else in either VPC can reach them. SSH and ICMP are never opened to 0.0.0.0/0.

## Why the App-B SSH source is the bastion private IP

With ProxyJump the laptop does not connect to App-B directly. The bastion opens a new SSH connection to App-B, so App-B sees the bastion's private IP (10.10.1.108) as the source, not my laptop's address. The login line `Last login ... from 10.10.1.108` in the terminal output confirms this.
