# Linux NAT Instance Configuration

## 1. Overview

The Amazon Linux 2023 EC2 instance was configured as a Network Address Translation (NAT) instance to provide outbound internet connectivity to resources in the private subnet.

The NAT implementation uses Linux IPv4 packet forwarding and `nftables` for address translation and traffic forwarding.

## 2. Network Details

| Parameter | Configuration |
|---|---|
| Operating System | Amazon Linux 2023 |
| Instance Role | Bastion and NAT |
| Private IP | `10.20.0.65` |
| Network Interface | `ens5` |
| Private Subnet | `10.20.0.128/25` |
| NAT Method | IP masquerading |
| Firewall Framework | `nftables` |

## 3. Prerequisites

Before configuring Linux NAT:

- The EC2 instance must be deployed in the public subnet.
- The public subnet must have a default route to the Internet Gateway.
- The private subnet must have a route to the NAT instance network interface.
- EC2 source/destination checking must be disabled on the NAT instance.
- Security group rules must permit the required traffic.

## 4. Enable IPv4 Forwarding

IPv4 forwarding allows the Linux instance to route packets between network interfaces or network paths.

The configuration was made persistent using:

```bash
sudo tee /etc/sysctl.d/99-nat.conf <<'EOF'
net.ipv4.ip_forward = 1
EOF
```

Apply the configuration:

```bash
sudo sysctl --system
```

Verify:

```bash
sysctl net.ipv4.ip_forward
```

Expected value:

```text
net.ipv4.ip_forward = 1
```

## 5. Install nftables

Install the required package:

```bash
sudo dnf install nftables -y
```

Check the service:

```bash
sudo systemctl status nftables
```

## 6. Configure NAT and Forwarding Rules

The NAT configuration uses IP masquerading for traffic originating from the private subnet.

The rules include:

- NAT masquerading for `10.20.0.128/25`.
- Forwarding private subnet traffic towards `ens5`.
- Allowing established and related return traffic.
- Restricting forwarding according to the configured firewall policy.

The configuration was stored in:

`/etc/nftables/main.nft`

The rules were loaded and tested using the nftables service.

## 7. Enable Persistent Configuration

The nftables service was enabled to load the firewall configuration during system startup.

The configuration file was included through:

`/etc/sysconfig/nftables.conf`

The service was restarted to validate that the rules loaded successfully:

```bash
sudo systemctl enable nftables
sudo systemctl restart nftables
sudo systemctl status nftables
```

Verify the active ruleset:

```bash
sudo nft list ruleset
```

## 8. AWS-Side NAT Configuration

The EC2 source/destination check was disabled on the Linux NAT instance.

This is required because the instance forwards traffic that is not necessarily addressed to or sourced from its own private IP address.

The private route table was configured with:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | Linux NAT instance network interface |

## 9. Validation

NAT functionality was validated using the private Windows Server.

### IP connectivity test

```cmd
ping 8.8.8.8
```

Result: Four packets sent, four received, zero packet loss.

### DNS resolution test

```cmd
nslookup google.com
```

Result: DNS resolution completed successfully.

Linux internet connectivity was also tested:

```bash
ping -c 4 google.com
```

Result: Four successful replies.

These tests confirmed outbound IP connectivity and DNS resolution during the lab.

## 10. Troubleshooting Considerations

If private instances cannot access the internet, verify:

1. EC2 source/destination checking is disabled on the NAT instance.
2. IPv4 forwarding is enabled.
3. The private route table points to the correct NAT network interface.
4. The public subnet has a default route to the Internet Gateway.
5. NAT masquerading rules are loaded.
6. Forwarding rules allow the required traffic.
7. Security groups and network ACLs permit the required traffic.
8. DNS configuration is functioning correctly.

## 11. Limitations

This implementation uses a self-managed Linux NAT instance. It requires operating system maintenance, firewall management, and availability monitoring.

For production environments, evaluate managed NAT Gateway services or a highly available NAT design.

---

**Configuration status:** Implemented and connectivity tested.

**Environment:** Personal AWS learning lab.
