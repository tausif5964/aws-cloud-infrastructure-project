# AWS Connectivity Validation and Test Results

## 1. Purpose

This document records the connectivity tests performed on the AWS public and private network infrastructure.

The objective was to verify SSH administration, private Windows RDP access, outbound internet connectivity, DNS resolution, and EC2 monitoring.

**Environment:** Personal AWS learning lab  
**AWS Region:** Asia Pacific (Mumbai) – `ap-south-1`

## 2. Test Environment

| Component | Configuration |
|---|---|
| VPC CIDR | `10.20.0.0/24` |
| Public subnet | `10.20.0.0/25` |
| Private subnet | `10.20.0.128/25` |
| Linux EC2 | Amazon Linux 2023 |
| Linux private IP | `10.20.0.65` |
| Windows EC2 | Windows Server 2025 |
| Windows private IP | `10.20.0.210` |
| NAT mechanism | Linux EC2 NAT instance |

## 3. SSH Connectivity Test

**Objective:** Verify administrative SSH access to the public Linux EC2 instance.

Command:

```powershell
ssh -i "Cloud-Lab-Key.pem" ec2-user@<LINUX_PUBLIC_IP>
```

**Expected result:** Successful SSH authentication and access to the Linux terminal.

**Observed result:** SSH access was successful.

**Status:** PASS

## 4. Private Windows RDP Connectivity Test

**Objective:** Verify that the private Windows Server can be accessed without assigning it a public IPv4 address.

SSH local port forwarding command:

```powershell
ssh -i "Cloud-Lab-Key.pem" -L 13389:10.20.0.210:3389 ec2-user@<LINUX_PUBLIC_IP>
```

Remote Desktop destination:

```text
localhost:13389
```

**Expected result:** The RDP client connects to the private Windows Server through the SSH tunnel.

**Observed result:** Windows Remote Desktop access through the SSH tunnel was successful.

**Status:** PASS

## 5. Windows Outbound IP Connectivity Test

**Objective:** Verify outbound IP connectivity from the private Windows Server through the Linux NAT instance.

Command:

```cmd
ping 8.8.8.8
```

**Observed result:**

- Packets sent: 4
- Packets received: 4
- Packet loss: 0%
- Approximate response time: 2 ms

**Status:** PASS

This test confirmed outbound ICMP connectivity to the target at the time of testing.

## 6. Windows DNS Resolution Test

**Objective:** Verify DNS resolution from the private Windows Server.

Command:

```cmd
nslookup google.com
```

**Observed result:**

- DNS server responded successfully.
- DNS server address: `10.20.0.2`
- `google.com` resolved successfully.

**Status:** PASS

## 7. Linux Internet Connectivity Test

**Objective:** Verify internet connectivity from the public Linux EC2 instance.

Command:

```bash
ping -c 4 google.com
```

**Observed result:** Four successful replies were received.

**Status:** PASS

## 8. CloudWatch Monitoring Validation

Amazon CloudWatch was used to review EC2 per-instance metrics.

The following metrics were reviewed:

| Metric | Purpose | Result |
|---|---|---|
| CPUUtilization | Review CPU usage | Reviewed |
| NetworkIn | Review incoming network traffic | Reviewed |
| NetworkOut | Review outgoing network traffic | Reviewed |

Basic EC2 monitoring was used.

**Status:** REVIEWED

## 9. Consolidated Test Summary

| Test ID | Test Description | Result |
|---|---|---|
| TC-01 | SSH access to public Linux EC2 | PASS |
| TC-02 | RDP through SSH local port forwarding | PASS |
| TC-03 | Windows outbound IP connectivity | PASS |
| TC-04 | Windows DNS resolution | PASS |
| TC-05 | Linux internet connectivity | PASS |
| TC-06 | CloudWatch CPU and network metrics review | REVIEWED |

## 10. Validation Conclusion

The connectivity tests confirmed that the implemented public and private network architecture supported the intended administrative and outbound connectivity paths during the lab.

The private Windows Server was accessed through SSH local port forwarding, and outbound connectivity was verified through the Linux NAT instance.

CloudWatch CPU and network metrics were also reviewed to demonstrate basic infrastructure monitoring.

These results represent testing performed during the lab and do not constitute continuous availability or production-readiness certification.

---

**Validation status:** Completed  
**Environment:** Personal AWS learning lab  
**Documentation:** Technical test record
