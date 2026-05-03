# AWS VPC (Virtual Private Cloud)

A **VPC** is your own logically isolated network within AWS. It gives you full control over IP ranges, subnets, routing, and network security. Every AWS account comes with a **default VPC** in each region.

---

## Core Concepts

### VPC
- Tied to a **single region**
- You define a **CIDR block** (e.g., `10.0.0.0/16`) — this is the IP range for your entire VPC
- Supports IPv4 and optionally IPv6
- Can have up to 5 VPCs per region (soft limit, can be increased)

### Subnets
Subnets divide your VPC's IP range into smaller segments. Each subnet lives in a **single Availability Zone**.

| Type | Description |
|---|---|
| **Public Subnet** | Has a route to an Internet Gateway; resources can be reached from the internet |
| **Private Subnet** | No direct internet route; used for databases, app servers, etc. |

> 💡 AWS **reserves 5 IP addresses** in every subnet (first 4 + last 1). A `/24` subnet gives you 251 usable IPs, not 256.

### Internet Gateway (IGW)
- Allows resources in **public subnets** to communicate with the internet
- Horizontally scaled, redundant, and highly available — no bandwidth bottleneck
- **One IGW per VPC**
- Must be attached to the VPC AND a route must exist in the subnet's route table

### Route Tables
- Every subnet is associated with a route table
- Route tables define where network traffic is directed
- A **main route table** is created automatically with your VPC
- Best practice: create **custom route tables** per subnet type

```
# Example public subnet route table
Destination     Target
10.0.0.0/16     local          ← VPC-internal traffic
0.0.0.0/0       igw-xxxxxxxx   ← all other traffic → Internet Gateway
```

---

## NAT — Private Subnet Internet Access

Private subnets can't reach the internet directly. A **NAT (Network Address Translation)** device lets them initiate outbound connections (e.g., downloading updates) without being reachable from the internet.

| | NAT Gateway | NAT Instance |
|---|---|---|
| **Managed by** | AWS | You |
| **Availability** | Highly available within AZ | Single EC2 instance |
| **Bandwidth** | Up to 100 Gbps | Limited by instance size |
| **Recommended** | ✅ Yes | ❌ Legacy, avoid |

> ⚠️ NAT Gateway is **AZ-specific**. For high availability, deploy one per AZ.

NAT Gateway lives in a **public subnet** and has an Elastic IP. Private subnet route table points `0.0.0.0/0` to the NAT Gateway.

```
# Private subnet route table
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       nat-xxxxxxxx   ← NAT Gateway (in public subnet)
```

---

## Security: Security Groups vs NACLs

These are the two layers of network security in a VPC — they work differently and complement each other.

### Security Groups
- Act as a **virtual firewall at the instance level**
- **Stateful** — if inbound traffic is allowed, the response is automatically allowed
- Rules are **allow-only** (no explicit deny)
- Evaluated **all at once** (not top-to-bottom)

```
# Example: web server security group
Inbound:  Allow TCP 443 from 0.0.0.0/0   (HTTPS)
Inbound:  Allow TCP 22 from 10.0.0.0/16  (SSH from VPC only)
Outbound: Allow all traffic               (default)
```

### Network ACLs (NACLs)
- Act as a **firewall at the subnet level**
- **Stateless** — return traffic must be explicitly allowed
- Support both **Allow** and **Deny** rules
- Rules are evaluated **in order** (lowest number first); first match wins
- Each subnet is associated with exactly one NACL

```
# Example NACL rules
Rule #  Type      Protocol  Port   Source        Action
100     HTTP      TCP       80     0.0.0.0/0     ALLOW
200     HTTPS     TCP       443    0.0.0.0/0     ALLOW
300     SSH       TCP       22     203.0.113.0   ALLOW
*       ALL       ALL       ALL    0.0.0.0/0     DENY   ← default
```

### Quick Comparison

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Evaluation | All rules | In order (rule #) |
| Default | Deny all inbound | Allow all |

---

## VPC Connectivity Options

### VPC Peering
- Connect **two VPCs** privately (same or different accounts/regions)
- Traffic stays on the AWS backbone — not over the internet
- **Not transitive**: if A↔B and B↔C, A cannot reach C through B
- No overlapping CIDR blocks allowed

### VPC Endpoints
Connect to AWS services **without leaving the AWS network** (no IGW, NAT, or public IP needed).

| Type | Description | Use Case |
|---|---|---|
| **Gateway Endpoint** | Free; added to route table | S3, DynamoDB |
| **Interface Endpoint** | Elastic Network Interface with private IP; costs money | Most other AWS services |

### AWS Transit Gateway
- Acts as a **central hub** to connect multiple VPCs and on-premises networks
- Solves the transitive peering problem
- Supports thousands of VPCs

### Site-to-Site VPN
- Encrypted connection between your **on-premises network** and AWS over the public internet
- Uses a **Virtual Private Gateway** (AWS side) and a **Customer Gateway** (on-prem side)
- Quick to set up; bandwidth limited by internet speed

### AWS Direct Connect
- **Dedicated physical connection** from your data center to AWS
- More consistent performance, lower latency, higher bandwidth than VPN
- Takes weeks to provision; more expensive
- Can be combined with VPN for encryption

---

## Elastic IP (EIP)
- A **static public IPv4 address** you own in your account
- Can be re-mapped to different instances (useful for failover)
- **Free while attached** to a running instance; charged when unattached or idle
- NAT Gateways require an EIP

---

## VPC Flow Logs
- Capture information about **IP traffic** going to/from network interfaces in your VPC
- Can be published to **CloudWatch Logs** or **S3**
- Useful for troubleshooting connectivity and security auditing
- Does **not** capture: DNS traffic, DHCP, traffic to the instance metadata service (`169.254.169.254`)

---

## Typical 3-Tier VPC Architecture

```
VPC: 10.0.0.0/16
│
├── Public Subnet (10.0.1.0/24)  — AZ-a
│   ├── Internet Gateway
│   ├── NAT Gateway
│   └── Load Balancer / Bastion Host
│
├── Private Subnet (10.0.2.0/24) — AZ-a
│   └── App Servers (EC2)
│
└── Private Subnet (10.0.3.0/24) — AZ-a
    └── Databases (RDS)
```

Replicate across multiple AZs for high availability.

---

## Key Exam/Interview Points

- A subnet = 1 AZ; a VPC spans all AZs in a region
- Default VPC has a public subnet in every AZ with a pre-configured IGW
- Security groups are **stateful**; NACLs are **stateless** — this is frequently tested
- VPC Peering is **non-transitive** — Transit Gateway solves this
- Gateway Endpoints (S3, DynamoDB) are **free**; Interface Endpoints cost money
- NAT Gateway is **not** free — it charges per hour + per GB processed
- `0.0.0.0/0` in a route table = default route (all traffic not matched by a more specific rule)
- Ephemeral ports (`1024–65535`) must be allowed in NACL outbound rules for stateless return traffic

---

*Last updated: May 2026*