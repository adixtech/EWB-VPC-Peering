# Cleanup and learnings

## What I deleted

1. Terminated ewb-bastion, ewb-app-a and ewb-app-b (`screenshots/31-cleanup-instances-terminated.png`). The only other instance listed (KaamLok-Prod...) is stopped and is not part of this project.
2. Deleted the peering connection ewb-a-b-peer. The console reported that 3 of 3 related routes were deleted (`screenshots/32-cleanup-peering-deleted.png`).
3. Deleted both lab VPCs. Only the account's default VPC remains (`screenshots/33-cleanup-vpcs-deleted.png`).

Cleanup was done on 23 October 2026, after all test evidence was saved.

## Remaining checks

Items I did not capture a screenshot for, and should confirm in the console (Mumbai region, filter by tag `Project = EWB-VPC-Peering`):

- EBS volumes and snapshots
- Elastic IPs (none were created)
- The `ewb-lab` key pair (delete it in EC2 if no longer needed, and keep the .pem file out of the repository)
- Billing page for any remaining charges (usage reporting can lag)

## What I learned

- A public subnet is defined by its route to an internet gateway, and an instance also needs a public IP to be reachable directly. Only the bastion had both.
- Peering is only a link. Routes on both sides and a security group rule on the destination are all needed for traffic to flow.
- Private servers can be administered safely through one bastion using SSH ProxyJump, without copying the private key onto any instance.
- Security groups are stateful and rules should be scoped to /32 sources.
- Public IPs that are not Elastic IPs change when an instance is stopped and started, so the SSH config entry for the bastion has to be updated when it changes.

## Problems I hit and fixed

- The first `ssh ewb-app-a` attempt failed with "Could not resolve hostname" (`screenshots/17-ssh-app-a.png`); the retry connected to App-A. (Add your own cause here, for example a config entry that was not saved yet.)
- The route table for the A-private subnet was first created with a wrong name (it looked like `ewb-rt-b-private`). I renamed it to `ewb-rt-a-private`.
- The bastion's public IP was different in different sessions (3.108.67.58, then 13.126.9.6), so the `HostName` in the SSH config had to match the current one.
