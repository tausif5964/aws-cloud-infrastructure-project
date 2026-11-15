# AWS Network Design

## 1. Network Architecture

The project uses a custom AWS Virtual Private Cloud (VPC) with a segmented public and private subnet architecture in the Mumbai region (`ap-south-1`).

The public subnet hosts an Amazon Linux 2023 EC2 instance configured for administrative access and NAT functionality. The private subnet hosts a Windows Server 2025 EC2 instance without a public IPv4 address.

## 2. IP Addressing Plan

| Network Component | CIDR / Address | Purpose |
|---|---|---|
| VPC | `10.20.0.0/24` | Overall network boundary |
| Public subnet | `10.20.0.0/25` | Linux bastion and NAT instance |
| Private subnet | `10.20.0.128/25` | Private Windows workload |
| Linux EC2 | `10.20.0.65` | Bastion and NAT |
| Windows EC2 | `10.20.0.210` | Private Windows Server |

## 3. Routing Design

### Public Route Table

| Destination | Target |
|---|---|
| `10.20.0.0/24` | Local |
| `0.0.0.0/0` | Internet Gateway |

The public route table enables internet routing for the public subnet through the attached Internet Gateway.

### Private Route Table

| Destination | Target |
|---|---|
| `10.20.0.0/24` | Local |
| `0.0.0.0/0` | Linux NAT instance network interface |

The private subnet sends outbound internet-bound traffic through the Linux NAT instance.

## 4. Internet Connectivity Design

The Linux EC2 instance performs network address translation using IPv4 forwarding and `nftables`.

The configuration includes:

- EC2 source/destination check disabled.
- IPv4 forwarding enabled.
- IP masquerading for the private subnet.
- Forwarding rules for permitted traffic.
- Persistent NAT configuration.

This design allows private instances to initiate outbound internet connections without requiring public IPv4 addresses.

## 5. Administrative Access Design

The private Windows Server is accessed through SSH local port forwarding.

The administrator establishes an SSH connection to the public Linux instance and forwards local TCP port `13389` to the private Windows Server's RDP port `3389`.

The RDP client connects to `localhost:13389`.

## 6. Security Boundaries

- Linux SSH access is restricted through a source IP-based security group rule.
- Windows RDP access is restricted to the permitted security group source.
- The Windows instance has no public IPv4 address.
- Security groups control instance-level inbound and outbound traffic.
- Private keys and credentials are excluded from the repository.

## 7. Design Limitations

This architecture is a personal learning lab, not a production deployment.

A production implementation should evaluate NAT high availability, managed NAT Gateway options, resilient bastion access, IAM least privilege, centralized logging, monitoring alerts, and infrastructure as code.

## 8. Architecture Validation

The network design was validated through:

- SSH access to the Linux instance.
- RDP access to Windows through SSH tunnelling.
- Successful outbound IP connectivity from Windows.
- Successful DNS resolution from Windows.
- CloudWatch EC2 CPU and network metric review.
