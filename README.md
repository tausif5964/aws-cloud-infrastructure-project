# AWS Cloud Infrastructure: Public and Private Network Architecture

**Project Type:** Hands-on AWS Infrastructure Engineering Lab  
**Cloud Provider:** Amazon Web Services (AWS)  
**AWS Region:** Asia Pacific (Mumbai) — `ap-south-1`  
**Primary Services:** Amazon VPC, EC2, Internet Gateway, Route Tables, Security Groups, Linux NAT, Windows Server, Amazon CloudWatch

---

## 1. Project Overview

This project demonstrates the design, deployment, configuration, and validation of a segmented AWS cloud network using public and private subnets.

The infrastructure consists of an Amazon Linux 2023 EC2 instance deployed in a public subnet and a Windows Server 2025 EC2 instance deployed in a private subnet.

The Linux instance was configured as a NAT instance to provide outbound internet connectivity to private subnet resources without assigning them public IP addresses.

Secure administrative access to the private Windows Server was established through SSH local port forwarding via the Linux instance. Amazon CloudWatch was used to review EC2 CPU and network metrics.

The project demonstrates practical skills in:

- AWS cloud infrastructure deployment
- VPC and subnet architecture
- Linux system administration
- Network Address Translation (NAT)
- Windows Server administration
- Secure remote connectivity
- Network troubleshooting
- Cloud infrastructure monitoring
- Resource lifecycle and cost management

## 2. Project Objectives

- Design and deploy a custom AWS Virtual Private Cloud (VPC).
- Create separate public and private subnets.
- Configure an Internet Gateway and subnet-specific route tables.
- Deploy and administer Linux and Windows EC2 instances.
- Configure a Linux-based NAT instance for private subnet outbound connectivity.
- Implement security group rules to restrict administrative access.
- Establish private Windows Server access through SSH tunnelling.
- Validate IP connectivity and DNS resolution.
- Monitor EC2 metrics using Amazon CloudWatch.
- Document implementation, testing, troubleshooting, security, and cleanup procedures.

## 3. Architecture Overview

![AWS Cloud Infrastructure Architecture](<architecture/AWS Cloud Network Architecture.png>)

The architecture uses a two-subnet network design to separate public-facing infrastructure from private workloads.

### Public Subnet

- Hosts an Amazon Linux 2023 EC2 instance.
- Provides SSH-based administrative access.
- Performs NAT for outbound traffic originating from the private subnet.
- Uses the Internet Gateway for internet connectivity.

### Private Subnet

- Hosts a Windows Server 2025 EC2 instance.
- Does not have a public IPv4 address assigned.
- Uses the Linux NAT instance for outbound internet connectivity.
- Allows administrative RDP access through SSH local port forwarding.

### Monitoring

- Amazon CloudWatch provides EC2 per-instance CPU and network metrics.

> **Architecture note:** The Linux EC2 instance is configured as a self-managed NAT instance. AWS NAT Gateway was not used in this project.

## 4. Network Design

| Component | Configuration |
|---|---|
| VPC CIDR | `10.20.0.0/24` |
| Public Subnet | `10.20.0.0/25` |
| Private Subnet | `10.20.0.128/25` |
| AWS Region | `ap-south-1` |
| Public Workload | Amazon Linux 2023 |
| Private Workload | Windows Server 2025 |
| NAT Mechanism | Linux EC2 NAT instance |
| Monitoring | Amazon CloudWatch |

### Traffic Flow

**Public instance connectivity**

`Internet ↔ Internet Gateway ↔ Public Subnet ↔ Linux EC2`

**Private instance outbound connectivity**

`Windows EC2 → Private Route Table → Linux NAT Instance → Internet Gateway → Internet`

**Private Windows administrative access**

`Administrator Workstation → SSH Tunnel → Linux EC2 → Private Windows Server (RDP)`

## 5. Implementation Summary

### 5.1 VPC and Subnet Configuration

- Created a custom VPC with a `/24` IPv4 CIDR block.
- Created one public and one private subnet.
- Enabled automatic public IPv4 assignment for the public subnet.
- Disabled automatic public IPv4 assignment for the private subnet.

### 5.2 Internet Gateway and Routing

- Created and attached an Internet Gateway to the custom VPC.
- Configured the public route table with a default route to the Internet Gateway.
- Configured the private route table with a default route pointing to the Linux NAT instance's network interface.

### 5.3 EC2 Deployment

- Deployed an Amazon Linux 2023 instance in the public subnet.
- Deployed a Windows Server 2025 instance in the private subnet.
- Configured security groups to control SSH and RDP access.

### 5.4 Linux NAT Configuration

The Linux instance was configured to forward and translate traffic originating from the private subnet.

Configuration included:

- Disabled EC2 source/destination checks.
- Enabled IPv4 packet forwarding.
- Configured IP masquerading using `nftables`.
- Configured forwarding rules for private subnet traffic.
- Persisted NAT configuration across service restarts.

### 5.5 Secure Windows Access Through SSH Tunnelling

Configured SSH local port forwarding through the Linux instance to access the private Windows Server over RDP.

**Example SSH command:**

Replace the key path and public IP placeholder with your own values.

```powershell
ssh -i "Cloud-Lab-Key.pem" -L 13389:10.20.0.210:3389 ec2-user@<LINUX_PUBLIC_IP>
```

Connect the Remote Desktop client to:

```text
localhost:13389
```

**Port forwarding details:**

| Parameter | Purpose |
|---|---|
| `13389` | Local port on the administrator's workstation |
| `10.20.0.210` | Private IP address of the Windows Server |
| `3389` | Remote Desktop Protocol (RDP) port |

The SSH session must remain active while using the RDP connection.

### 5.6 CloudWatch Monitoring

Reviewed EC2 per-instance metrics using Amazon CloudWatch, including:

- `CPUUtilization`
- `NetworkIn`
- `NetworkOut`

Basic monitoring was used for this lab. Detailed monitoring was not enabled.

## 6. Validation and Test Results

The following connectivity and access tests were performed during implementation.

| Test | Target | Result |
|---|---|---|
| SSH access | Public Linux EC2 | Successful |
| RDP through SSH tunnel | Private Windows EC2 | Successful |
| IP connectivity | Windows → `8.8.8.8` | Successful |
| DNS resolution | Windows → `google.com` | Successful |
| Internet connectivity | Linux EC2 | Successful |
| CloudWatch metrics | EC2 instances | Metrics reviewed |

### Windows Connectivity Evidence

**IP connectivity**

- Command: `ping 8.8.8.8`
- Packets sent: 4
- Packets received: 4
- Packet loss: 0%

**DNS resolution**

- Command: `nslookup google.com`
- Result: DNS resolution successful.

These results validate the tested connectivity paths at the time of the lab.

## 7. Security Considerations

The following security controls were implemented during the lab:

- Deployed the Windows Server without a public IPv4 address.
- Routed administrative access to the private Windows instance through the Linux bastion/NAT instance.
- Restricted SSH access using a source IP-based security group rule.
- Restricted RDP access to the permitted security group source.
- Used security groups to control instance-level network access.
- Excluded private keys and credentials from the repository.

**Production considerations**

This project is a learning and portfolio lab, not a production-ready reference architecture.

Production deployments should additionally evaluate:

- High availability and fault tolerance.
- Managed NAT Gateway options.
- IAM least-privilege policies.
- Centralized logging and monitoring.
- Operating system patch management.
- Hardened administrative access controls.
- Infrastructure as Code (IaC).
- Backup and disaster recovery strategies.

## 8. Project Documentation

The following documents provide additional technical details.

| Document | Description |
|---|---|
| [Network Design](architecture/network-design.md) | VPC, subnet design, network layout, and traffic flow |
| [Implementation Guide](documentation/implementation-guide.md) | Infrastructure deployment and implementation steps |
| [Linux NAT Configuration](documentation/linux-nat-configuration.md) | Linux packet forwarding, `nftables`, and NAT configuration |
| [Security Configuration](documentation/security-configuration.md) | Security groups and administrative access controls |
| [Connectivity Validation](documentation/connectivity-validation.md) | Connectivity tests and validation results |
| [Troubleshooting](documentation/troubleshooting.md) | Issues encountered and troubleshooting procedures |
| [Cost and Cleanup](documentation/cost-and-cleanup.md) | Lab cost awareness and resource cleanup |

## 9. Repository Structure

```text
aws-cloud-infrastructure-project/
│
├── README.md
├── .gitignore
│
├── architecture/
│   ├── AWS Cloud Network Architecture.png
│   └── network-design.md
│
├── documentation/
│   ├── implementation-guide.md
│   ├── linux-nat-configuration.md
│   ├── security-configuration.md
│   ├── connectivity-validation.md
│   ├── troubleshooting.md
│   └── cost-and-cleanup.md
│
└── Screenshoots/
    ├── 01-VPC-Details.png
    ├── 02-Subnets.png
    ├── 03-Subnets.png
    ├── 04-Internet-Gateway..png
    ├── 05-Public-Route-Table.png
    ├── 06-Private-Route-Table.png
    ├── 07A-SG-Public-EC2.png
    ├── 07B-SG-Windows-Private.png
    ├── 08-Linux-EC2-Details.png
    ├── 09-Linux-SSH-System-Details.png
    ├── 10-Windows-Private-Network.png
    ├── 11-Windows-RDP-Through-Bastion.png
    ├── 12A--CloudWatch-NetworkIn.png
    └── 12B-CloudWatch-NetworkIn.png
```

## 10. Implementation Evidence

The following screenshots provide visual evidence of the AWS infrastructure configuration, deployment, connectivity testing, and monitoring.

### 10.1 VPC and Network Architecture

- [VPC Details](Screenshoots/01-VPC-Details.png)
- [Subnet Configuration](Screenshoots/02-Subnets.png)
- [Additional Subnet Details](Screenshoots/03-Subnets.png)
- [Internet Gateway](Screenshoots/04-Internet-Gateway..png)
- [Public Route Table](Screenshoots/05-Public-Route-Table.png)
- [Private Route Table](Screenshoots/06-Private-Route-Table.png)

### 10.2 Security Configuration

- [Public EC2 Security Group](Screenshoots/07A-SG-Public-EC2.png)
- [Private Windows Security Group](Screenshoots/07B-SG-Windows-Private.png)

### 10.3 EC2 Infrastructure

- [Linux EC2 Instance Details](Screenshoots/08-Linux-EC2-Details.png)
- [Linux SSH and System Details](Screenshoots/09-Linux-SSH-System-Details.png)
- [Windows Private Network Configuration](Screenshoots/10-Windows-Private-Network.png)

### 10.4 Secure Remote Access

- [Windows RDP Through SSH Bastion Tunnel](Screenshoots/11-Windows-RDP-Through-Bastion.png)

### 10.5 CloudWatch Monitoring

- [CloudWatch Monitoring Evidence A](Screenshoots/12A--CloudWatch-NetworkIn.png)
- [CloudWatch Monitoring Evidence B](Screenshoots/12B-CloudWatch-NetworkIn.png)

## 11. Cost Management and Cleanup

The infrastructure was deployed as a temporary hands-on lab.

After implementation and validation, the AWS resources were deleted as part of the cleanup process.

This project also emphasizes:

- Cost awareness when deploying cloud resources.
- Resource lifecycle management.
- Reviewing service usage and estimated costs.
- Cleaning up temporary infrastructure after testing.

> Before publishing screenshots, verify that sensitive information such as account identifiers, public IP addresses, credentials, private keys, and other identifying details are removed or masked.

## 12. Key Learning Outcomes

Through this project, I gained practical experience with:

- AWS VPC and subnet architecture.
- Public and private network segmentation.
- Route table and Internet Gateway configuration.
- Linux system administration and packet forwarding.
- NAT configuration using `nftables`.
- Windows Server deployment and remote administration.
- SSH tunnelling and private instance access.
- Network connectivity and DNS troubleshooting.
- EC2 monitoring through CloudWatch.
- Cloud resource cleanup and cost awareness.

---

**Project Status:** Implementation and connectivity validation completed.  
**Environment:** Personal AWS learning lab.  
**Documentation:** Technical documentation and implementation evidence included in this repository.
