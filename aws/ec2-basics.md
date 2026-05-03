# AWS EC2 (Elastic Compute Cloud)

**EC2** provides resizable virtual servers (instances) in the cloud. It is the backbone of most AWS architectures and one of the most heavily tested services on AWS exams.

---

## Instance Basics

### AMI (Amazon Machine Image)
An AMI is a template used to launch an EC2 instance. It includes:
- The operating system
- Pre-installed software and configurations
- Storage mappings (root volume + additional volumes)

| AMI Type | Description |
|---|---|
| **AWS Managed** | Amazon Linux 2, Ubuntu, Windows Server, etc. |
| **Marketplace** | Third-party pre-configured images |
| **Custom** | AMIs you create from your own instances |

AMIs are **region-specific** — you must copy an AMI to use it in another region.

### Instance Types
Instance types determine CPU, memory, network, and storage capacity. They follow a naming convention:

```
m6i.xlarge
│ │  └── Size  (nano, micro, small, medium, large, xlarge, 2xlarge...)
│ └──── Generation (6 = 6th gen)
└────── Family (m = general purpose)
```

#### Instance Families

| Family | Purpose | Examples |
|---|---|---|
| **m** | General purpose — balanced CPU/RAM | m6i, m7g |
| **c** | Compute optimized — high CPU | c6i, c7g |
| **r** | Memory optimized — high RAM | r6i, r7g |
| **t** | Burstable — cost-efficient for variable workloads | t3, t4g |
| **i** | Storage optimized — high IOPS NVMe | i3, i4i |
| **p / g** | GPU — ML training, graphics | p4, g5 |
| **inf** | Inferentia — ML inference | inf2 |

> 💡 **Tip:** "t" instances have a **CPU credit** system. They earn credits when idle and spend them when bursting. Good for dev/test, bad for sustained high CPU.

---

## Pricing Models

Choosing the right pricing model is critical for cost optimization — a common exam scenario.

### On-Demand
- Pay by the **hour or second** with no commitment
- Most flexible, most expensive
- Best for: unpredictable workloads, short-term projects, testing

### Reserved Instances (RI)
- Commit to 1 or 3 years → up to **72% discount** vs On-Demand
- Payment options: All Upfront, Partial Upfront, No Upfront
- Types:
  - **Standard RI** — biggest discount, cannot change instance family
  - **Convertible RI** — smaller discount, can change instance family/OS

### Savings Plans
- More flexible than RIs — commit to a **$/hour spend** for 1 or 3 years
- Applies automatically across instance families, regions, and even Lambda/Fargate
- Two types: **Compute Savings Plans** (most flexible) and **EC2 Instance Savings Plans**

### Spot Instances
- Use AWS's **spare capacity** — up to **90% discount**
- Can be **interrupted** by AWS with 2-minute warning when capacity is reclaimed
- Best for: batch jobs, data analysis, fault-tolerant workloads
- **Not suitable** for databases, critical apps, or anything that can't tolerate interruption

### Dedicated Hosts
- A **physical server** dedicated to you
- Required for certain software licenses (Oracle, Windows Server per-core licensing)
- Most expensive option
- Can be On-Demand or Reserved

### Dedicated Instances
- Instances run on hardware dedicated to your account
- May share the physical host with other instances **from your account**
- Cheaper than Dedicated Hosts, but less control

#### Pricing Comparison

| Model | Discount | Commitment | Best For |
|---|---|---|---|
| On-Demand | — | None | Short-term, unpredictable |
| Reserved | Up to 72% | 1–3 years | Steady-state workloads |
| Savings Plans | Up to 66% | 1–3 years | Flexible steady-state |
| Spot | Up to 90% | None | Fault-tolerant, batch |
| Dedicated Host | — | Optional | License compliance |

---

## Storage Options

### EBS (Elastic Block Store)
- **Network-attached** block storage — like a virtual hard drive
- Persists independently of the instance lifecycle (by default)
- **AZ-locked** — must be in the same AZ as the instance
- Snapshots can be taken to S3 (cross-region copy supported)

#### EBS Volume Types

| Type | Use Case | Max IOPS |
|---|---|---|
| **gp3** | General purpose SSD (default) | 16,000 |
| **gp2** | Older general purpose SSD | 16,000 |
| **io2 / io2 Block Express** | High-performance databases | 256,000 |
| **st1** | Throughput-optimized HDD (big data, logs) | 500 MB/s |
| **sc1** | Cold HDD — cheapest, infrequent access | 250 MB/s |

> 💡 **gp3 is preferred over gp2** — you can independently configure IOPS and throughput without paying more.

### Instance Store
- **Physically attached** storage (NVMe SSDs on the host machine)
- Extremely fast — ideal for temp files, caches, buffers
- **Ephemeral** — data is lost when the instance stops or terminates
- Cannot be detached or snapshotted

### EFS (Elastic File System)
- **Managed NFS** — can be mounted by **multiple instances simultaneously**
- Great for shared storage across instances or AZs
- Automatically scales; pay per GB used
- More expensive than EBS; Linux only

| | EBS | Instance Store | EFS |
|---|---|---|---|
| Type | Block | Block | File |
| Persistence | Yes | ❌ No | Yes |
| Multi-attach | Limited | No | ✅ Yes |
| Cross-AZ | No | No | ✅ Yes |

---

## Networking

### Elastic IP (EIP)
- Static public IPv4 address attached to an instance
- Survives instance stop/start (unlike regular public IPs)
- Free while attached to a running instance; charged when idle

### ENI (Elastic Network Interface)
- A virtual network card you can attach to instances
- Each instance gets a primary ENI by default
- You can attach additional ENIs (useful for network appliances or dual-homed setups)

### Enhanced Networking
- Uses SR-IOV for higher bandwidth, lower latency, lower CPU overhead
- Required for high-throughput workloads
- Enabled via **ENA (Elastic Network Adapter)** or **Intel 82599 VF**

### Placement Groups
Control how instances are physically placed on hardware:

| Type | Behavior | Best For |
|---|---|---|
| **Cluster** | Same rack, same AZ — lowest latency | HPC, big data (high network throughput) |
| **Spread** | Different hardware — max 7 instances per AZ | Critical instances that must not fail together |
| **Partition** | Groups of instances on separate racks | Hadoop, Kafka, Cassandra |

---

## EC2 Lifecycle

```
Pending → Running → Stopping → Stopped → Terminated
                  ↘ Rebooting ↗
```

- **Stop**: Instance shuts down; EBS data preserved; no compute charge; EBS still charged
- **Terminate**: Instance deleted; root EBS deleted by default (configurable)
- **Hibernate**: RAM saved to EBS, instance "paused" — fast resume, preserves in-memory state
- **Reboot**: No data loss; same as an OS restart

---

## User Data & Metadata

### User Data
- A script that runs **once at first launch** (by default)
- Used to bootstrap instances (install packages, configure apps, etc.)
- Runs as root; accessible at `http://169.254.169.254/latest/user-data`

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2</h1>" > /var/www/html/index.html
```

### Instance Metadata
- Information about the running instance available at:
  `http://169.254.169.254/latest/meta-data/`
- Includes: instance ID, AMI ID, public IP, IAM role credentials, AZ, etc.
- **IMDSv2** (recommended) requires a session token for access — more secure than IMDSv1

---

## Auto Scaling

### Auto Scaling Group (ASG)
- Automatically **adds or removes EC2 instances** based on demand
- Works with a **Launch Template** (or older Launch Configuration) that defines instance settings
- Integrates with **Elastic Load Balancer** for health checks and traffic distribution

#### Scaling Policies

| Policy | Description |
|---|---|
| **Target Tracking** | Maintain a metric at a target value (e.g., CPU at 50%) — simplest |
| **Step Scaling** | Scale by defined amounts based on CloudWatch alarm thresholds |
| **Scheduled Scaling** | Scale at specific times (e.g., scale up every Monday 8am) |
| **Predictive Scaling** | ML-based forecast of future traffic |

#### ASG Key Settings
- **Min / Desired / Max** capacity
- **Health check grace period** — time to wait before checking new instance health
- **Cooldown period** — pause between scaling actions to let metrics stabilize

---

## Load Balancers (ELB)

ASGs pair with Elastic Load Balancers to distribute traffic:

| Type | Layer | Best For |
|---|---|---|
| **ALB** (Application) | Layer 7 (HTTP/HTTPS) | Web apps, microservices, path/host-based routing |
| **NLB** (Network) | Layer 4 (TCP/UDP) | Ultra-low latency, static IP, gaming, IoT |
| **GLB** (Gateway) | Layer 3 | Third-party network appliances (firewalls, IDS) |
| **CLB** (Classic) | Layer 4/7 | Legacy only — avoid for new architectures |

### ALB Features
- Path-based routing: `/api/*` → target group A, `/images/*` → target group B
- Host-based routing: `api.example.com` vs `app.example.com`
- Sticky sessions (via cookie)
- Native HTTP/2 and WebSocket support
- Returns a **fixed DNS name** (not a static IP)

---

## Key Exam/Interview Points

- **EBS is AZ-locked**; use snapshots to move data across AZs or regions
- **Instance Store is ephemeral** — any data is lost on stop/terminate
- **Spot Instances can be interrupted** — never use for stateful or critical workloads
- **gp3 > gp2** — cheaper and more configurable
- **Cluster Placement Group** = lowest latency but single point of AZ failure
- **Spread Placement Group** = max 7 instances per AZ
- **User data runs once** at first boot (unless explicitly configured otherwise)
- **IMDSv2 is more secure** than IMDSv1 — always prefer it
- **ALB operates at Layer 7**; **NLB at Layer 4** — very frequently tested
- Reserved Instances apply to a running instance; **Savings Plans are more flexible**
- **Hibernate** preserves RAM to EBS; the instance must have an encrypted root volume

---

*Last updated: May 2026*