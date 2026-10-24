# Concepts explained

These answers use the results from my own lab.

## 1. Why is the return route needed?

A ping needs two journeys: the request and the reply. App-A at 10.10.2.105 can only send to 10.20.1.108 because ewb-rt-a-private has 10.20.0.0/16 pointing at the peering connection. App-B then has to send its reply back to 10.10.2.105, and it can only do that if ewb-rt-b-private has 10.10.0.0/16 pointing at the same connection. Without that route the reply has nowhere to go and the sender sees 100% loss, even though the peering says Active. Peering does not add routes automatically, so each side must have its own.

## 2. Why is the App-B SSH source the bastion private IP?

I reach App-B with ProxyJump. My laptop connects to the bastion, and then the bastion opens a separate SSH connection to App-B. App-B therefore sees the bastion's private address, 10.10.1.108, as the client. My laptop's IP never reaches App-B, so the rule on ewb-sg-app-b must allow 10.10.1.108/32. The `Last login ... from 10.10.1.108` line in my App-B session shows this.

## 3. Why does an Active peering not guarantee a successful ping?

An Active peering is only the link between the two VPCs. Traffic still needs a route table entry on each side that points the other VPC's CIDR at the peering connection, and a security group rule on the destination that allows the traffic (here ICMP Echo Request). In my lab the peering was Active in `20-peering-active.png`, but pings only worked after I added the routes. In the controlled failure, removing only the ICMP rule made the ping fail again while the peering stayed Active.

## 4. Can VPC B use VPC A's internet gateway over peering?

No. VPC peering is one-to-one and non-transitive. It only carries traffic between the two VPCs' own address ranges. VPC B has no internet gateway and its route table has no 0.0.0.0/0 route, so App-B has no internet access, even though VPC A has an internet gateway. This is also why the private servers cannot download packages.

## Extra: why use private IPs for all tests?

Peering routes only private CIDRs (10.10.0.0/16 and 10.20.0.0/16), and App-A and App-B have no public IP. Testing with private IPs proves the peering and routes are what make the connection work: the same ping failed before peering and succeeded after.
