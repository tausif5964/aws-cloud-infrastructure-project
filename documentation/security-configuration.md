# AWS Security Configuration

## 1. Overview

Security groups were configured to control network access to the EC2 instances deployed in the custom VPC.

The design separates public administrative access from private workload access. The Windows Server instance does not have a public IPv4 address and is accessed through SSH local port forwarding via the Linux instance.

## 2. Security Design

The security configuration follows these principles:

- Restrict administrative access to approved sources.
- Avoid direct public exposure of the private Windows Server.
- Use security group references where appropriate for instance-to-instance access.
- Permit only the network traffic required by the lab.
- Keep credentials and private keys outside the repository.

## 3. Security Groups

The project uses separate security groups for the Linux and Windows workloads.

### 3.1 SG-Public-EC2

**Purpose:** Controls access to the public Linux EC2 instance.

- SSH access is restricted to the administrator's public IP address using a `/32` source rule.
- Required NAT traffic from the private subnet is permitted.
- Outbound traffic is permitted according to the configured security group rules.

### 3.2 SG-Private-EC2

**Purpose:** Controls access to the private Linux-related workload traffic where applicable.

- SSH access is restricted to the permitted public EC2 security group source.
- Other access is controlled according to the configured rules.

### 3.3 SG-Windows-Private

**Purpose:** Controls access to the private Windows Server.

- RDP access on TCP port `3389` is permitted from the designated public EC2 security group.
- The Windows Server has no public IPv4 address.
- Administrative RDP access is established through the SSH tunnel.

## 4. Administrative Access Flow

The administrator connects to the public Linux instance using SSH.

An SSH local port-forwarding tunnel is then established to the private Windows Server.

Example:

```powershell
ssh -i "Cloud-Lab-Key.pem" -L 13389:10.20.0.210:3389 ec2-user@<LINUX_PUBLIC_IP>
```

The Remote Desktop client connects to:

`localhost:13389`

This avoids exposing the Windows RDP service directly to the public internet.

## 5. Network-Level Security

The following controls were implemented:

| Security Control | Implementation |
|---|---|
| Private Windows public IP | Not assigned |
| SSH access | Restricted by source IP |
| RDP access | Restricted by security group source |
| Private subnet internet access | Through Linux NAT |
| EC2 NAT source/destination check | Disabled on NAT instance |
| Network segmentation | Separate public and private subnets |

## 6. Credential and Repository Safety

The following sensitive items must not be committed to GitHub:

- `.pem` private key files.
- AWS access keys and secret keys.
- Passwords and decrypted Windows administrator credentials.
- Local environment files containing secrets.
- Account identifiers or sensitive screenshots.

A `.gitignore` file is included to help prevent accidental publication of private keys and common sensitive files.

Screenshots should be reviewed and redacted before publication.

## 7. Security Limitations

This project is a personal learning lab and does not represent a complete production security architecture.

Additional production controls may include:

- IAM least-privilege policies and role-based access.
- AWS Systems Manager Session Manager.
- Centralized logging and audit trails.
- Security monitoring and alerting.
- Regular operating system patching.
- Hardened bastion access.
- High availability and recovery planning.
- Network ACL review and segmentation.

## 8. Validation

The security design was validated through:

- SSH access to the public Linux instance.
- RDP access to the private Windows Server through SSH port forwarding.
- Verification that the Windows Server had no public IPv4 address.
- Successful private subnet outbound connectivity through the Linux NAT instance.

**Configuration status:** Implemented and tested as part of the personal AWS infrastructure lab.
