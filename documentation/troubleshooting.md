# AWS Infrastructure Troubleshooting Guide

## 1. Purpose

This document records troubleshooting scenarios and diagnostic approaches relevant to the AWS public and private network infrastructure lab.

It covers Linux NAT, private subnet connectivity, routing, SSH tunnelling, Windows networking, and DNS resolution.

## 2. Troubleshooting Methodology

The following approach was used when diagnosing connectivity issues:

1. Identify the affected instance and network path.
2. Check the instance's network configuration.
3. Verify subnet associations and route tables.
4. Review security group rules.
5. Check Linux packet forwarding and NAT configuration.
6. Test IP connectivity independently from DNS resolution.
7. Review service status and relevant logs.
8. Repeat connectivity tests after corrective action.

## 3. Linux NAT Troubleshooting

### Scenario 1: Private instance cannot access the internet

**Possible causes:**
- EC2 source/destination checking is enabled on the NAT instance.
- IPv4 forwarding is disabled.
- Private route table points to an incorrect target.
- NAT masquerading rules are missing or inactive.
- Security group or network ACL rules block traffic.

**Diagnostic commands:**

```bash
sysctl net.ipv4.ip_forward
sudo nft list ruleset
sudo systemctl status nftables
ip addr show ens5
```

**Resolution approach:**

- Verify source/destination checking is disabled.
- Enable IPv4 forwarding.
- Check the private subnet's default route.
- Verify masquerading and forwarding rules.
- Review security groups and network ACLs.

### Scenario 2: NAT rules do not persist after restart

**Possible causes:**
- nftables service is disabled.
- Configuration file is not included in the service configuration.
- Rules contain syntax errors.

**Diagnostic commands:**

```bash
sudo systemctl status nftables
sudo systemctl is-enabled nftables
sudo nft list ruleset
```

**Resolution approach:**

- Enable the nftables service.
- Verify the configuration file path.
- Confirm the configuration is included in the nftables startup configuration.
- Restart the service and verify the loaded ruleset.

## 4. Private Windows Connectivity Troubleshooting

### Scenario 3: RDP connection to private Windows Server fails

**Possible causes:**
- SSH tunnel is not active.
- Local port is incorrect.
- Windows RDP service is unavailable.
- Security group does not allow TCP 3389 from the permitted source.
- Windows instance or network configuration is incorrect.

**Diagnostic checks:**

- Verify SSH connectivity to the Linux instance.
- Keep the SSH tunnel session active.
- Confirm the tunnel forwards local port `13389` to `10.20.0.210:3389`.
- Verify Windows RDP availability.
- Review the Windows security group.

**Correct RDP destination:**

```text
localhost:13389
```

## 5. DNS and IP Connectivity Troubleshooting

### Scenario 4: IP connectivity works but domain names do not resolve

**Possible causes:**
- DNS server configuration is incorrect.
- DNS traffic is blocked.
- DNS resolution service is unavailable.

**Diagnostic commands:**

```cmd
ipconfig /all
nslookup google.com
```

**Resolution approach:**

- Verify the DNS server address.
- Check DNS-related security group and network ACL rules.
- Confirm that the configured DNS resolver is reachable.
- Test DNS resolution again.

### Scenario 5: Private Windows Server cannot reach an external IP

**Diagnostic command:**

```cmd
ping 8.8.8.8
```

**Possible causes:**
- Private route table has an incorrect default route.
- NAT instance forwarding is disabled.
- NAT masquerading is not functioning.
- Security group or network ACL rules restrict outbound traffic.

**Resolution approach:**

Check the private route table, NAT instance configuration, forwarding rules, and network security controls.

## 6. SSH Tunnel Diagnostics

### Scenario 6: SSH local port forwarding does not work

**Possible causes:**
- Incorrect SSH command.
- Incorrect private Windows IP address.
- Local port `13389` is already in use.
- SSH session was disconnected.
- RDP service is unavailable on the Windows instance.

**Tunnel command:**

```powershell
ssh -i "Cloud-Lab-Key.pem" -L 13389:10.20.0.210:3389 ec2-user@<LINUX_PUBLIC_IP>
```

**Resolution approach:**

- Verify the key path and Linux public IP.
- Confirm the private Windows IP.
- Check local port availability.
- Keep the SSH session running.
- Verify RDP availability from the Linux instance.

## 7. General AWS Network Checks

When troubleshooting AWS connectivity, review:

- EC2 instance state and network interface details.
- VPC and subnet CIDR configuration.
- Route table associations.
- Internet Gateway attachment.
- Security group inbound and outbound rules.
- Network ACL inbound and outbound rules.
- EC2 source/destination checking.
- Linux forwarding and NAT configuration.

## 8. Troubleshooting Summary

| Area | Diagnostic Focus |
|---|---|
| Linux NAT | IPv4 forwarding and nftables |
| Private routing | Default route and NAT network interface |
| SSH | Authentication and tunnel status |
| Windows RDP | TCP 3389 and RDP availability |
| DNS | Resolver configuration and lookup |
| Internet access | IP connectivity and NAT |
| AWS networking | Route tables and security groups |

## 9. Conclusion

This troubleshooting guide provides a structured approach to diagnosing common connectivity problems in a segmented AWS network.

It reflects practical troubleshooting knowledge across Linux administration, Windows Server networking, AWS routing, NAT, SSH tunnelling, and DNS.

**Environment:** Personal AWS learning lab.

**Note:** This document includes diagnostic scenarios and recommended checks. Not every scenario represents an incident that occurred during the lab.
