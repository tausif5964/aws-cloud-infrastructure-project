# AWS Cloud Infrastructure: Public and Private Network Architecture

**Project Type:** Hands-on AWS Infrastructure Engineering Lab  
**Cloud Provider:** Amazon Web Services (AWS)  
**AWS Region:** Asia Pacific (Mumbai) — `ap-south-1`  
**Services:** Amazon VPC, EC2, Internet Gateway, Security Groups, Route Tables, Linux NAT, Windows Server, Amazon CloudWatch

---

## 1. Project Overview

This project demonstrates the design, deployment, configuration, and validation of a segmented AWS cloud network containing public and private subnets.

The infrastructure was designed to host a publicly accessible Linux EC2 instance and a private Windows Server EC2 instance. The Linux instance was configured to provide Network Address Translation (NAT), enabling outbound internet connectivity for resources in the private subnet without assigning them public IP addresses.

Secure administrative access to the private Windows Server was established through SSH-based port forwarding via the Linux instance. Amazon CloudWatch was used to review EC2 performance and network metrics.

The project focuses on practical cloud infrastructure administration, network segmentation, secure remote access, connectivity troubleshooting, and infrastructure monitoring.

## 2. Project Objectives

- Design and deploy a custom AWS Virtual Private Cloud (VPC).
- Create separate public and private subnets.
- Configure an Internet Gateway and subnet-specific route tables.
- Deploy and administer Linux and Windows EC2 instances.
- Configure a Linux-based NAT instance for private subnet outbound connectivity.
- Implement security group rules to restrict administrative access.
- Access a private Windows Server through SSH tunnelling.
- Validate IP connectivity and DNS resolution.
- Monitor EC2 metrics using Amazon CloudWatch.
- Document implementation, testing, security considerations, and cleanup.

## 3. Architecture Overview

![AWS Cloud Infrastructure Architecture](<architecture/AWS Cloud Network Architecture.png>)

The infrastructure follows a two-subnet network design.

### Public Subnet

- Hosts an Amazon Linux 2023 EC2 instance.
- Provides SSH-based administrative access.
- Performs NAT for outbound traffic from the private subnet.
- Uses the Internet Gateway for internet connectivity.

### Private Subnet

- Hosts a Windows Server 2025 EC2 instance.
- Does not have a public IPv4 address assigned.
- Uses the Linux NAT instance for outbound internet connectivity.
- Allows administrative RDP access through SSH local port forwarding.

### Monitoring

- Amazon CloudWatch provides EC2 CPU and network performance metrics.

> **Architecture note:** The Linux EC2 instance is configured as a NAT instance. This project does not use AWS NAT Gateway.

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

**Public instance**

Internet ↔ Internet Gateway ↔ Public Subnet ↔ Linux EC2

**Private instance outbound traffic**

Windows EC2 → Private Route Table → Linux NAT Instance → Internet Gateway → Internet

**Private Windows administration**

Administrator Workstation → SSH Tunnel → Linux EC2 → Private Windows Server (RDP)

## 5. Implementation Summary

### 5.1 VPC and Subnet Configuration

- Created a custom VPC with a `/24` IPv4 CIDR block.
- Created one public and one private subnet.
- Enabled automatic public IPv4 assignment for the public subnet.
- Kept automatic public IPv4 assignment disabled for the private subnet.

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

- Disabling EC2 source/destination checks.
- Enabling IPv4 packet forwarding.
- Configuring IP masquerading using `nftables`.
- Configuring forwarding rules for private subnet traffic.
- Persisting the NAT configuration across service restarts.

### 5.5 Secure Windows Access Through SSH Tunnelling

Configured SSH local port forwarding through the Linux instance to reach the private Windows Server over RDP.

Example command (replace the key path and public IP with your own values):

```powershell
ssh -i "Cloud-Lab-Key.pem" -L 13389:10.20.0.210:3389 ec2-user@<LINUX_PUBLIC_IP>
```

The RDP client then connects to:

```text
localhost:13389
```

The SSH session must remain active while using the RDP connection.

### 5.6 CloudWatch Monitoring

Reviewed EC2 per-instance metrics using Amazon CloudWatch, including:

- `CPUUtilization`
- `NetworkIn`
- `NetworkOut`

Basic monitoring was used for this lab. Detailed monitoring was not enabled.

## 6. Validation and Test Results

The following tests were performed during implementation.

| Test | Target | Result |
|---|---|---|
| SSH access | Public Linux EC2 | Successful |
| RDP through SSH tunnel | Private Windows EC2 | Successful |
| IP connectivity | Windows → `8.8.8.8` | Successful |
| DNS resolution | Windows → `google.com` | Successful |
| Internet connectivity | Linux EC2 | Successful |
| CloudWatch metrics | EC2 instances | Metrics reviewed |

### Windows Connectivity Evidence

- `ping 8.8.8.8`: 4 packets sent, 4 received, 0% packet loss.
- `nslookup google.com`: DNS resolution successful.

These results validate the tested connectivity paths at the time of the lab.

## 7. Security Considerations

- The Windows Server was deployed without a public IPv4 address.
- Administrative access to the private Windows instance was routed through the Linux bastion/NAT instance.
- SSH access was restricted using a source IP-based security group rule.
- RDP access was restricted to the permitted security group source.
- Security groups were used to control instance-level network access.
- Private keys and credentials are not included in this repository.

**Production considerations:** This is a learning and portfolio lab, not a production-ready reference architecture. Production deployments should additionally evaluate high availability, managed NAT Gateway options, IAM least privilege, centralized logging, patch management, hardened access controls, and infrastructure as code.

## 8. Documentation

The following documents provide additional technical details about the project.

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
└── screenshots/
    ├── 01-VPC-Details.png
    ├── 02-Subnets.png
    ├── 03-Subnets.png
    ├── 04-Internet-Gateway..png
    ├── 05-Public-Route-Table.png
    ├── 06-Private-Route-Table.png
    ├── 07A-SG-Public-EC2.png
    ├── 07B-SG-Windows-Private.png
    ├── 08-Linux-EC2-Details.png
    ├── 09-Windows-EC2-Details.png
    ├── 10-Linux-SSH-System-Details.png
    ├── 11-Windows-Private-Network.png
    ├── 12-Windows-RDP-Through-Bastion.png
    ├── 13A--CloudWatch-NetworkIn.png
    └── 13B-CloudWatch-NetworkIn.png
```

## 10. Implementation Evidence

The following screenshots provide visual evidence of the AWS infrastructure configuration, deployment, connectivity testing, and monitoring.

### 10.1 VPC and Network Architecture

- [VPC Details](screenshots/01-VPC-Details.png)
- [Subnet Configuration](screenshots/02-Subnets.png)
- [Additional Subnet Details](screenshots/03-Subnets.png)
- [Internet Gateway](screenshots/04-Internet-Gateway..png)
- [Public Route Table](screenshots/05-Public-Route-Table.png)
- [Private Route Table](screenshots/06-Private-Route-Table.png)

### 10.2 Security Configuration

- [Public EC2 Security Group](screenshots/07A-SG-Public-EC2.png)
- [Private Windows Security Group](screenshots/07B-SG-Windows-Private.png)

### 10.3 EC2 Infrastructure

- [Linux EC2 Instance Details](screenshots/08-Linux-EC2-Details.png)
- [Windows EC2 Instance Details](screenshots/09-Windows-EC2-Details.png)
- [Linux SSH and System Details](screenshots/10-Linux-SSH-System-Details.png)
- [Windows Private Network Configuration](screenshots/11-Windows-Private-Network.png)

### 10.4 Secure Remote Access

- [Windows RDP Through SSH Bastion Tunnel](screenshots/12-Windows-RDP-Through-Bastion.png)

### 10.5 CloudWatch Monitoring

- [CloudWatch Monitoring Evidence A](screenshots/13A--CloudWatch-NetworkIn.png)
- [CloudWatch Monitoring Evidence B](screenshots/13B-CloudWatch-NetworkIn.png)

## 11. Cost Management and Cleanup

The infrastructure was deployed as a temporary hands-on lab. After implementation and validation, the AWS resources were deleted as part of the cleanup process.

The project emphasizes:

- Cost awareness when deploying cloud resources.
- Resource lifecycle management.
- Reviewing service usage and estimated costs.
- Cleaning up temporary infrastructure after testing.

Before publishing screenshots, sensitive information such as account identifiers, public IP addresses, credentials, private keys, and other identifying details should be removed or masked.

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
**Documentation:** Project documentation and implementation evidence included in this repository.
