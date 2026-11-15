# AWS Cloud Infrastructure – Implementation Guide

## 1. Purpose

This document describes the implementation process for the AWS public and private network architecture lab.

The environment was deployed in the AWS Mumbai region (`ap-south-1`) using Amazon VPC, EC2, Internet Gateway, route tables, security groups, Linux NAT, Windows Server, and CloudWatch.

## 2. Implementation Sequence

The infrastructure was implemented in the following stages:

1. Create a custom VPC.
2. Create public and private subnets.
3. Attach an Internet Gateway.
4. Configure public and private route tables.
5. Deploy the Linux EC2 instance.
6. Deploy the private Windows Server EC2 instance.
7. Configure security groups.
8. Configure Linux as a NAT instance.
9. Establish private Windows access through SSH port forwarding.
10. Validate connectivity and review CloudWatch metrics.

## 3. VPC and Subnet Deployment

Created a custom VPC with the IPv4 CIDR block:

`10.20.0.0/24`

Two subnets were configured:

| Subnet | CIDR | Purpose |
|---|---|---|
| Public | `10.20.0.0/25` | Linux bastion and NAT |
| Private | `10.20.0.128/25` | Windows Server |

Automatic public IPv4 assignment was enabled for the public subnet and disabled for the private subnet.

## 4. Internet Gateway and Route Tables

An Internet Gateway was created and attached to the custom VPC.

### Public routing

The public route table contains:
- VPC local route: `10.20.0.0/24`
- Default route: `0.0.0.0/0` through the Internet Gateway

### Private routing

The private route table contains:
- VPC local route: `10.20.0.0/24`
- Default route: `0.0.0.0/0` through the Linux NAT instance network interface

## 5. EC2 Deployment

### Public Linux instance

- Operating system: Amazon Linux 2023
- Instance type: `t3.micro`
- Private IP: `10.20.0.65`
- Role: Bastion host and NAT instance

### Private Windows instance

- Operating system: Windows Server 2025 Base
- Instance type: `t3.small`
- Private IP: `10.20.0.210`
- Role: Private Windows workload
- Public IPv4: Not assigned

## 6. Linux NAT Configuration

The Amazon Linux instance was configured to provide NAT functionality for the private subnet.

The implementation included:

- Disabling EC2 source/destination checking.
- Enabling IPv4 forwarding.
- Installing and configuring `nftables`.
- Configuring IP masquerading for private subnet traffic.
- Configuring forwarding rules.
- Persisting the firewall rules across service restarts.

The private subnet's default route was configured to use the Linux instance's network interface.

Detailed configuration is documented in:

`linux-nat-configuration.md`

## 7. Secure Windows Administration

The private Windows Server was accessed using SSH local port forwarding through the Linux instance.

Example command:

```powershell
ssh -i "Cloud-Lab-Key.pem" -L 13389:10.20.0.210:3389 ec2-user@<LINUX_PUBLIC_IP>
```

The Remote Desktop client connects to:

`localhost:13389`

The SSH session must remain active while using the tunnel.

## 8. Connectivity Validation

The following validation tests were completed:

| Validation | Result |
|---|---|
| SSH access to Linux | Successful |
| RDP access through SSH tunnel | Successful |
| Windows outbound IP connectivity | Successful |
| Windows DNS resolution | Successful |
| Linux internet connectivity | Successful |
| CloudWatch CPU and network metrics review | Completed |

Windows connectivity was tested using:

```cmd
ping 8.8.8.8
nslookup google.com
```

The ping test returned four successful replies with zero packet loss. DNS resolution for `google.com` was also successful.

## 9. CloudWatch Monitoring

Amazon CloudWatch was used to review EC2 per-instance metrics.

Metrics reviewed:

- CPUUtilization
- NetworkIn
- NetworkOut

Basic EC2 monitoring was used for this lab. Detailed monitoring and additional alerting were outside the project scope.

## 10. Completion and Cleanup

The implementation and connectivity validation were completed.

The temporary AWS resources were deleted after testing as part of the lab cleanup process.

## 11. Scope and Limitations

This project demonstrates practical AWS infrastructure administration in a personal learning environment.

It does not implement production-grade high availability, automated infrastructure provisioning, centralized logging, or a managed NAT Gateway.

These areas are potential improvements for a future version of the architecture.
