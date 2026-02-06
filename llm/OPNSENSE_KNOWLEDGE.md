# OPNsense Configuration Knowledge Base

> **Purpose:** LLM-optimized reference for configuring OPNsense firewalls via MCP tools.
> **Usage:** Include this file in your LLM's context alongside the OPNsense MCP server.

---

## Quick Reference

### Default Credentials & Ports
| Item | Value |
|------|-------|
| Default login | `root` / `opnsense` |
| Web UI | `https://<ip>:443` (or `:8443` if changed) |
| SSH | Port 22 (disabled by default) |
| API endpoint | `https://<ip>/api/` |

### Interface Naming
| OPNsense Name | Typical Use | Example |
|---------------|-------------|---------|
| `wan` | Internet uplink | DHCP or static from ISP |
| `lan` | Internal network | 192.168.1.1/24 |
| `optN` | Additional interfaces | opt1, opt2, etc. for VLANs/DMZ |

---

## MCP Tool Mapping

Use these MCP tools for OPNsense operations:

### Connection & Status
```
opnsense_test_connection      - Verify API connectivity
opnsense_get_interfaces       - List all interfaces with status
opnsense_system_get_settings  - Get system-level settings
```

### Firewall Rules
```
opnsense_list_firewall_rules  - List all rules
opnsense_get_firewall_rule    - Get rule details by UUID
opnsense_create_firewall_rule - Create new rule
opnsense_update_firewall_rule - Modify existing rule
opnsense_delete_firewall_rule - Remove rule
opnsense_toggle_firewall_rule - Enable/disable rule
opnsense_find_firewall_rules  - Search by description
opnsense_create_firewall_preset - Quick rule from template (allow-web, allow-ssh, block-all)
```

### VLANs
```
opnsense_list_vlans    - List all VLANs
opnsense_get_vlan      - Get VLAN details by tag
opnsense_create_vlan   - Create new VLAN (interface, tag, description)
opnsense_update_vlan   - Update VLAN description
opnsense_delete_vlan   - Remove VLAN
```

### NAT
```
opnsense_nat_list_outbound       - List outbound NAT rules
opnsense_nat_list_port_forwards  - List port forwards
opnsense_nat_get_mode            - Get NAT mode (automatic/hybrid/manual)
opnsense_nat_set_mode            - Set NAT mode
opnsense_nat_create_outbound_rule - Create outbound NAT
opnsense_nat_create_port_forward  - Create port forward
opnsense_nat_delete_outbound_rule - Delete outbound NAT
opnsense_nat_delete_port_forward  - Delete port forward
opnsense_nat_apply_changes        - Apply NAT configuration
```

### DHCP & DNS
```
opnsense_list_dhcp_leases    - List all DHCP leases
opnsense_find_device_by_name - Find device by hostname
opnsense_find_device_by_mac  - Find device by MAC address
opnsense_list_dns_blocklist  - List blocked domains
opnsense_block_domain        - Add domain to blocklist
opnsense_unblock_domain      - Remove from blocklist
```

### HAProxy (Load Balancer)
```
opnsense_haproxy_service_control - Start/stop/restart HAProxy
opnsense_haproxy_backend_list    - List backends
opnsense_haproxy_backend_create  - Create backend
opnsense_haproxy_frontend_list   - List frontends
opnsense_haproxy_frontend_create - Create frontend
opnsense_haproxy_stats           - Get statistics
```

### SSH/CLI Access (Advanced)
```
opnsense_ssh_execute          - Run arbitrary command
opnsense_ssh_show_pf_rules    - Show packet filter rules
opnsense_ssh_show_routing     - Display routing table
opnsense_ssh_reload_firewall  - Reload firewall rules
opnsense_ssh_system_status    - Comprehensive system status
```

### Backup & Maintenance
```
opnsense_create_backup   - Create config backup
opnsense_list_backups    - List available backups
opnsense_restore_backup  - Restore from backup
```

---

## Configuration Workflows

### 1. Initial Setup (Fresh Install)

```
1. opnsense_test_connection         → Verify API access
2. opnsense_get_interfaces          → Identify WAN/LAN interfaces
3. opnsense_list_firewall_rules     → Review default rules
4. opnsense_create_backup           → Backup before changes
```

### 2. Create a VLAN Network

```
1. opnsense_create_vlan
   - interface: "igc3" (physical interface)
   - tag: "10" (VLAN ID)
   - description: "Guest Network"

2. Assign interface in UI (API limitation)
   → Interfaces > Assignments > Add VLAN

3. opnsense_create_firewall_rule
   - interface: "opt1" (the new VLAN interface)
   - action: "pass"
   - source: "opt1 net"
   - destination: "any"
   - description: "Allow Guest to Internet"

4. opnsense_create_firewall_rule
   - interface: "opt1"
   - action: "block"
   - source: "opt1 net"
   - destination: "lan net"
   - description: "Block Guest to LAN"
   (Note: Place BEFORE the allow rule - rule order matters!)
```

### 3. Port Forward (Expose Internal Service)

```
1. opnsense_nat_create_port_forward
   - interface: "wan"
   - protocol: "tcp"
   - destination_port: "443"
   - target: "192.168.1.50"
   - local_port: "443"
   - description: "HTTPS to webserver"

2. opnsense_nat_apply_changes

3. Firewall rule is auto-created, verify with:
   opnsense_list_firewall_rules
```

### 4. Site-to-Site VPN (WireGuard)

```
1. Create WireGuard instance (UI required for key generation)
   → VPN > WireGuard > Instances

2. Add peer configuration
   → VPN > WireGuard > Peers

3. opnsense_create_firewall_rule
   - interface: "wg0"
   - action: "pass"
   - source: "wg0 net"
   - destination: "lan net"
   - description: "Allow WireGuard to LAN"

4. Add static route if needed
   → System > Routes > Configuration
```

### 5. Block Ads/Malware (DNS Blocklist)

```
1. opnsense_block_domain
   - domain: "ads.example.com"
   
Or use categories:
2. opnsense_apply_blocklist_category
   - category: "ads" | "malware" | "adult" | "social"
```

---

## Firewall Rule Logic

### Rule Order (Critical!)
Rules are processed **top to bottom, first match wins**.

```
Typical order:
1. Block rules (specific threats)
2. Allow rules (specific services)
3. Inter-VLAN rules
4. Default deny (implicit or explicit)
```

### Rule Parameters

| Parameter | Required | Values | Notes |
|-----------|----------|--------|-------|
| `action` | Yes | pass, block, reject | reject sends ICMP unreachable |
| `interface` | Yes | wan, lan, opt1, etc. | Where traffic enters |
| `direction` | Yes | in, out | Usually "in" |
| `protocol` | Yes | any, tcp, udp, icmp | |
| `source` | Yes | any, IP, network, alias | Use CIDR: 192.168.1.0/24 |
| `destination` | Yes | any, IP, network, alias | |
| `sourcePort` | No | port or range | For tcp/udp |
| `destinationPort` | No | port or range | For tcp/udp |
| `enabled` | No | true/false | Default: true |
| `description` | No | string | Always add for clarity |

### Common Patterns

**Allow LAN to Internet:**
```json
{
  "action": "pass",
  "interface": "lan",
  "direction": "in",
  "protocol": "any",
  "source": "lan net",
  "destination": "any"
}
```

**Block IoT to LAN:**
```json
{
  "action": "block",
  "interface": "opt1",
  "direction": "in",
  "protocol": "any",
  "source": "opt1 net",
  "destination": "lan net",
  "description": "Isolate IoT from LAN"
}
```

**Allow specific port:**
```json
{
  "action": "pass",
  "interface": "wan",
  "direction": "in",
  "protocol": "tcp",
  "source": "any",
  "destination": "wan address",
  "destinationPort": "443",
  "description": "Allow HTTPS inbound"
}
```

---

## NAT Concepts

### Outbound NAT Modes

| Mode | Use Case |
|------|----------|
| Automatic | Default, handles most cases |
| Hybrid | Auto rules + custom rules |
| Manual | Full control, must define all rules |

### When to Use Manual NAT
- Multi-WAN setups
- Policy-based routing
- VPN traffic that shouldn't be NATed
- Specific source IP requirements

### No-NAT Rules (Important!)
For inter-VLAN routing or VPN traffic that should keep original source IP:
```json
{
  "interface": "wan",
  "source_net": "10.0.6.0/24",
  "destination_net": "10.0.0.0/24",
  "nonat": "1",
  "description": "DMZ to LAN - no NAT"
}
```

---

## Troubleshooting Commands

Via `opnsense_ssh_execute`:

```bash
# Check firewall rules (pf)
pfctl -sr                    # Show all rules
pfctl -ss                    # Show state table
pfctl -si                    # Show statistics

# Network diagnostics
netstat -rn                  # Routing table
ifconfig -a                  # All interfaces
tcpdump -i wan -n port 443   # Capture traffic

# Service control
configctl webgui restart     # Restart web UI
configctl filter reload      # Reload firewall
configctl interface reload   # Reload interfaces

# Logs
clog /var/log/filter.log     # Firewall log
tail -f /var/log/system.log  # System log
```

---

## Common Mistakes

1. **Rule order wrong** - Block rules must come BEFORE allow rules
2. **Forgetting NAT** - Port forwards need corresponding firewall rules
3. **Interface confusion** - Rules apply to traffic ENTERING the interface
4. **VLAN not assigned** - Creating VLAN doesn't assign it to an interface
5. **DNS not set** - Clients need DNS server (usually the firewall IP)
6. **Gateway not set** - DHCP clients need gateway (firewall IP on that subnet)

---

## Security Best Practices

1. **Change default password immediately**
2. **Disable SSH unless needed** - Use API instead
3. **Enable HTTPS redirect** - System > Settings > Administration
4. **Create backups before changes** - `opnsense_create_backup`
5. **Use aliases for IP groups** - Easier rule management
6. **Log denied traffic** - Enable logging on block rules
7. **Keep firmware updated** - System > Firmware

---

## API Limitations (Requires Web UI)

Some operations cannot be done via API alone:

- Initial interface assignment
- WireGuard key generation
- Certificate creation/import
- Some plugin configurations
- System updates/firmware

For these, guide the user to the specific UI location.

---

## Quick Diagnostics

When something doesn't work:

```
1. opnsense_ssh_execute "pfctl -sr | grep <interface>"
   → Are the rules actually loaded?

2. opnsense_ssh_execute "pfctl -ss | grep <ip>"
   → Is there a state entry?

3. opnsense_ssh_execute "tcpdump -i <interface> host <ip>"
   → Is traffic reaching the firewall?

4. opnsense_list_firewall_rules
   → Check rule order and enabled status

5. opnsense_ssh_execute "netstat -rn"
   → Is the routing correct?
```
