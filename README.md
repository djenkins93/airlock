# Airlock

**Software airgap utility for Linux systems**

Airlock provides instant, multi-layered network isolation for scenarios requiring immediate disconnection from all networks. Designed for security researchers, journalists, privacy advocates, and anyone needing reliable on-demand network isolation.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-Linux-lightgrey.svg)
![Shell](https://img.shields.io/badge/shell-bash-green.svg)

## Overview

Airlock creates a software-defined airgap by simultaneously engaging multiple isolation layers, making network communication effectively impossible without explicit restoration. Unlike simply unplugging a cable or disabling Wi-Fi, Airlock provides defense-in-depth against accidental reconnection, rogue processes, and hardware-level network restoration attempts.

```
┌─────────────────────────────────────────────────────────────┐
│                     AIRLOCK ISOLATION LAYERS                │
├─────────────────────────────────────────────────────────────┤
│  Layer 1   │  Network process termination (vault mode)      │
│  Layer 2   │  Wireless radio kill (rfkill)                  │
│  Layer 3   │  DNS flush and resolver removal                │
│  Layer 4   │  Firewall rules (iptables/ip6tables)           │
│  Layer 5   │  Blackhole routing (IPv4 + IPv6)               │
│  Layer 6   │  Interface state control (link down)           │
│  Layer 7   │  Kernel module unloading (paranoid mode)       │
│  Layer 8   │  Module load prevention (vault mode)           │
│  Layer 9   │  USB network device blocking (vault mode)      │
│  Layer 10  │  Network cache clearing (ARP, routing)         │
│  Layer 11  │  Memory/clipboard sanitization (vault mode)    │
└─────────────────────────────────────────────────────────────┘
```

## Features

- **Instant activation** — Single command engages all isolation layers
- **Three isolation modes** — Standard, Paranoid, and Vault for escalating security needs
- **Multi-init support** — Works with systemd, runit, and OpenRC
- **Automatic recovery** — Restores full network functionality on unseal
- **Watchdog timer** — Optional auto-unseal after configurable timeout
- **Traffic monitoring** — Real-time display of blocked connection attempts
- **Comprehensive logging** — Full audit trail of all actions
- **Interactive and CLI modes** — Menu-driven or scriptable operation

## Installation

### Quick Install

```bash
# Clone the repository
git clone https://github.com/yourusername/airlock.git
cd airlock

# Install to system
sudo install -m 755 airlock.sh /usr/local/bin/airlock

# Optional: Install man page
sudo install -m 644 airlock.1 /usr/local/share/man/man1/
sudo mandb
```

### Dependencies

**Required:**
- `bash` (4.0+)
- `iproute2` (provides `ip` command)
- `iptables`

**Optional (enhanced functionality):**
- `rfkill` — Wireless radio control
- `conntrack` — Connection tracking flush
- `xclip` or `wl-copy` — Clipboard sanitization

### Distribution-Specific Installation

**Debian/Ubuntu:**
```bash
sudo apt install iproute2 iptables rfkill
```

**Fedora/RHEL:**
```bash
sudo dnf install iproute iptables rfkill
```

**Arch Linux:**
```bash
sudo pacman -S iproute2 iptables rfkill
```

**Void Linux:**
```bash
sudo xbps-install -S iproute2 iptables rfkill
```

**Alpine Linux:**
```bash
sudo apk add iproute2 iptables rfkill
```

**Gentoo:**
```bash
sudo emerge sys-apps/iproute2 net-firewall/iptables net-wireless/rfkill
```

## Usage

### Basic Commands

```bash
# Seal the airlock (block all network)
sudo airlock seal

# Unseal and restore network
sudo airlock unseal

# Check current status
sudo airlock status

# Interactive menu
sudo airlock
```

### Isolation Modes

**Standard Mode** — Firewall, routing, interfaces, DNS, and wireless radios:
```bash
sudo airlock seal
```

**Paranoid Mode** — Standard + kernel module unloading:
```bash
sudo airlock seal --paranoid
```

**Vault Mode** — Maximum isolation with anti-restoration measures:
```bash
sudo airlock seal --vault
```

### Watchdog Timer

Auto-unseal after a timeout (useful for remote administration):
```bash
# Default 5-minute watchdog
sudo airlock seal --watchdog

# Custom timeout (10 minutes)
sudo airlock seal --watchdog=600
```

### Monitoring

```bash
# Live blocked traffic monitor
sudo airlock monitor

# View blocked connection summary
sudo airlock log
```

### Recovery

```bash
# Force network repair (if unseal fails)
sudo airlock repair
```

## Modes Explained

| Feature | Standard | Paranoid | Vault |
|---------|:--------:|:--------:|:-----:|
| Firewall blocking | ✓ | ✓ | ✓ |
| Blackhole routing | ✓ | ✓ | ✓ |
| Wireless radio kill | ✓ | ✓ | ✓ |
| Interface shutdown | ✓ | ✓ | ✓ |
| DNS removal | ✓ | ✓ | ✓ |
| Cache clearing | ✓ | ✓ | ✓ |
| **Kernel module unload** | | ✓ | ✓ |
| **Network daemon kill** | | | ✓ |
| **USB network blocking** | | | ✓ |
| **Module load prevention** | | | ✓ |
| **Memory sanitization** | | | ✓ |

**When to use each mode:**

- **Standard** — Quick isolation for routine privacy needs, temporary offline work
- **Paranoid** — Handling sensitive data, malware analysis, security research
- **Vault** — Maximum security scenarios, forensic work, air-gapped computation

## Use Cases

### Security Research
Analyze malware samples or suspicious files without risk of network callbacks or data exfiltration.

### Sensitive Document Handling
Work with confidential documents knowing no process can transmit data externally.

### Privacy-Critical Operations
Generate cryptographic keys, manage passwords, or handle secrets in complete isolation.

### Incident Response
Immediately isolate a potentially compromised system while preserving state for analysis.

### Offline Development
Ensure build processes use only local dependencies with no external fetches.

### Travel Security
Quick isolation when connecting to untrusted networks or crossing borders.

## Files

| Path | Description |
|------|-------------|
| `/var/run/airlock.state` | Current seal state and configuration |
| `/var/log/airlock/airlock.log` | Action audit log |
| `/var/log/airlock/blocked.log` | Blocked connection attempts |
| `/etc/modprobe.d/airlock-block.conf` | Module blacklist (vault mode) |
| `/etc/udev/rules.d/99-airlock-block-usb-net.rules` | USB blocking (vault mode) |

## Troubleshooting

### Network won't restore after unseal

```bash
# Run repair mode
sudo airlock repair

# If repair fails, manually reload your NIC driver
sudo modprobe <your_driver>  # e.g., iwlwifi, e1000e, r8169

# Bring interface up
sudo ip link set <interface> up

# Request DHCP lease
sudo dhcpcd <interface>
```

### Finding your network driver

```bash
# List loaded network modules
lsmod | grep -E 'e1000|iwl|r8169|ath|rtl|mt7'

# Or check your interface
ls -la /sys/class/net/*/device/driver
```

### Checking seal status

```bash
# View detailed status
sudo airlock status

# Check state file directly
cat /var/run/airlock.state
```

## Platform Compatibility

Airlock is designed for broad Linux compatibility:

| Distribution | Init System | Status |
|-------------|-------------|--------|
| Void Linux | runit | ✓ Tested |
| Arch Linux | systemd | ✓ Compatible |
| Debian/Ubuntu | systemd | ✓ Compatible |
| Fedora/RHEL | systemd | ✓ Compatible |
| Alpine Linux | OpenRC | ✓ Compatible |
| Gentoo | OpenRC/systemd | ✓ Compatible |
| NixOS | systemd | ✓ Compatible |

**Requirements:**
- Linux kernel 3.10+ (for network namespace support)
- Root privileges (or appropriate capabilities)

## Security Considerations

- Airlock is a **software** solution and cannot prevent physical hardware attacks
- A determined attacker with physical access could potentially bypass isolation
- For true airgap requirements, combine with physical disconnection
- Vault mode provides the strongest software-based isolation available
- Always verify seal status before handling sensitive material

## Contributing

Contributions are welcome. Please ensure any changes:

1. Maintain cross-distribution compatibility
2. Include appropriate error handling
3. Update documentation as needed
4. Follow existing code style

## License

MIT License — See [LICENSE](LICENSE) file for details.

## Author

Developed for Project Symbiote — Privacy-focused security tooling for the modern threat landscape.

---

*"In space, the airlock is your last line of defense. In cyberspace, it's your first."*
