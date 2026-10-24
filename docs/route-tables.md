# Route tables

Each table is explicitly associated with its subnet (the "Explicit subnet associations" column in the screenshots).

## ewb-rt-a-public (rtb-049e653e2a995ca2b), associated with ewb-a-public

| Destination | Target | Purpose |
|---|---|---|
| 10.10.0.0/16 | local | Bastion to App-A inside VPC A |
| 0.0.0.0/0 | igw-088353fc4cb5b4bff | Internet access for the bastion |
| 10.20.0.0/16 | pcx-00d385cdc737a48a7 | Bastion SSH to App-B (ProxyJump) |

Evidence: `screenshots/21-rt-a-public-final.png`. Before peering: `screenshots/04a-rt-a-public.png`.

## ewb-rt-a-private (rtb-0e91f5a2ab1f3bbc6), associated with ewb-a-private

| Destination | Target | Purpose |
|---|---|---|
| 10.10.0.0/16 | local | VPC A internal traffic |
| 10.20.0.0/16 | pcx-00d385cdc737a48a7 | Forward path: App-A to VPC B |

Evidence: `screenshots/22-rt-a-private-final.png` (edit routes page) and `screenshots/21-rt-a-public-final.png` (list showing the table and its subnet association). Before peering: `screenshots/04b-rt-a-private.png`.

Note: in the two early screenshots (`04a`, `04b`, taken before peering) this table still had a wrong name that read like `ewb-rt-b-private`. I renamed it to `ewb-rt-a-private`, and the later screenshots show the correct name. The route table ID `rtb-0e91f5a2ab1f3bbc6` is the same in all of them.

## ewb-rt-b-private (rtb-0d898c941c3ab77c3), associated with ewb-b-private

| Destination | Target | Purpose |
|---|---|---|
| 10.20.0.0/16 | local | VPC B internal traffic |
| 10.10.0.0/16 | pcx-00d385cdc737a48a7 | Return path to VPC A |

Evidence: `screenshots/23-rt-b-private-final.png`. Before peering (local route only): `screenshots/08-rt-b-before-peering.png`.

## Why every table needs its route

- **Forward path:** App-A (10.10.2.105) sends to 10.20.1.108. The A-private table sends 10.20.0.0/16 to the peering connection.
- **Return path:** App-B replies to 10.10.2.105. The B-private table sends 10.10.0.0/16 to the same peering connection. Without it the replies are dropped and the ping fails even though the peering is Active.
- **Bastion route:** with ProxyJump the bastion itself opens the SSH connection to App-B, so the public subnet's table also needs the 10.20.0.0/16 route. It is more specific than 0.0.0.0/0, so it wins over the internet gateway route.
