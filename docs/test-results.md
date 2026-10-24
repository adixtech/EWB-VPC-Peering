# Test results

All tests were run from my own laptop (Windows PowerShell with OpenSSH, ProxyJump through the bastion). Private IPs: App-A 10.10.2.105, App-B 10.20.1.108, bastion 10.10.1.108.

| # | Test | Command and location | Expected | Actual | Result | Evidence |
|---|---|---|---|---|---|---|
| 01 | Baseline, before peering | `ping -c 4 10.20.1.108` on App-A | no replies | 4 transmitted, 0 received, 100% packet loss | Pass | `screenshots/18-baseline-ping-fail.png`, `terminal-output/01-baseline.txt` |
| 02 | Forward path, after routes | `ping -c 4 10.20.1.108` on App-A | 4 replies, 0% loss | 4 received, 0% packet loss, avg 0.297 ms | Pass | `screenshots/24-ping-forward-a-to-b.png`, `terminal-output/02-forward-a-to-b.txt` |
| 03 | Reverse path, after routes | `ping -c 4 10.10.2.105` on App-B | 4 replies, 0% loss | 4 received, 0% packet loss, avg 0.179 ms | Pass | `screenshots/25-ping-reverse-b-to-a.png`, `terminal-output/03-reverse-b-to-a.txt` |
| 04 | Private SSH to App-B | `ssh ewb-app-b` on laptop | shell on App-B through bastion | shell on `ip-10-20-1-108`, `hostname -I` shows 10.20.1.108 | Pass | `screenshots/26-ssh-app-b.png`, `terminal-output/04-ssh-app-b.txt` |
| 05 | Isolation | EC2 instance details for App-A and App-B | no public IPv4 | Public IPv4 address shows `-` for both | Pass | `screenshots/13-app-a-details.png`, `screenshots/14-app-b-details.png` |

## Other observations

- SSH to the bastion and to App-A worked before peering because both are in VPC A and use its local route (`screenshots/16-ssh-bastion.png`, `screenshots/17-ssh-app-a.png`). The first attempt in screenshot 17 failed with "Could not resolve hostname ewb-app-a"; the second attempt connected to App-A.
- In `screenshots/24-ping-forward-a-to-b.png`, a ping from App-A to the bastion's private IP (10.10.1.108) shows 100% loss. This is expected: ewb-sg-bastion only allows SSH, not ICMP.
- Peering status: Pending acceptance (`screenshots/19-peering-pending.png`), then Active after acceptance (`screenshots/20-peering-active.png`).

## Controlled failure

| Step | What I did | Result | Evidence |
|---|---|---|---|
| 1 | Removed only the ICMP Echo Request rule from ewb-sg-app-b, keeping the SSH rule | rule list shows only SSH from 10.10.1.108/32 | `screenshots/27-failure-rule-removed.png` |
| 2 | Pinged 10.20.1.108 from App-A | 4 transmitted, 0 received, 100% packet loss (run twice) | `screenshots/28-failure-ping-fails-a-to-b.png` |
| 3 | Restored the Echo Request rule from 10.10.2.105/32 | SSH and ICMP rules both present | `screenshots/29-rule-restored.png` |
| 4 | Ping after restoring the rule | App-B to App-A ping: 4 received, 0% loss | `screenshots/30-recovery-ping-b-to-a.png` |

Why the ping stopped: the route tables and the Active peering were unchanged, so packets still reached App-B. With the Echo Request rule gone, App-B's security group dropped the ICMP packets, so no replies came back. This shows a working route is not enough on its own; the security group on the destination must also allow the traffic.

Honest note on step 4: the recovery ping I captured was run from App-B to App-A, not the original App-A to App-B direction used in step 2. The restored rule is visible in screenshot 29, but I did not capture the App-A to App-B ping after the rule was restored.
