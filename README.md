# Pentesting Lab Setup: Kali Linux on a VirtualBox NAT Network

My first cybersecurity project: building an isolated virtual lab for practicing penetration testing safely, and troubleshooting the network problems that came up along the way.

## Project Overview

The goal was to build a contained environment where I can run offensive security tools without touching my home or school network. The lab uses VirtualBox, Kali Linux as the attacker machine, and a custom NAT Network so lab machines can talk to each other and reach the internet for updates.

## Lab Environment

| Component | Details |
|---|---|
| Host OS | Windows 11] |
| Hypervisor | Oracle VirtualBox 7.2.22 |
| Attacker VM | Kali Linux 2026.2 |
| Network type | VirtualBox NAT Network |
| Subnet | 10.0.0.0/24 |
| Gateway | 10.0.0.1 |
| Kali IP | 10.0.0.4 (static) |

## Network Design

<img width="588" height="265" alt="Screenshot 2026-10-27 120813" src="https://github.com/user-attachments/assets/e1ff3464-f863-4642-a9d8-82479722f204" />


Why a NAT Network instead of plain NAT or Bridged:

- **NAT Network**: VMs can reach each other and the internet, but outside devices cannot reach them. This is the right balance for a lab.
- **Plain NAT**: gives internet access, but VMs are isolated from each other.
- **Bridged**: exposes VMs directly on the real network, which is risky when running attack tools.
- **Host-only / Internal**: no internet, useful for fully isolated malware or exploit practice.

## Setup Steps

1. Installed VirtualBox on the host.
2. Created a NAT Network in **File > Tools > Network Manager > NAT Networks** with prefix `10.0.0.0/24`.
3. Imported/created the Kali Linux VM.
4. Set the VM's **Adapter 1** to **NAT Network** and selected the lab network by name.
5. Configured Kali's network settings (see below).
6. Verified connectivity with `ping`.

### Static IP configuration (NetworkManager)

```bash
sudo nmcli connection add type ethernet ifname eth0 con-name labnet \
  ipv4.method manual ipv4.addresses 10.0.0.4/24 \
  ipv4.gateway 10.0.0.1 ipv4.dns 8.8.8.8
sudo nmcli connection up labnet
```

### Static IP configuration (manual, temporary)

```bash
sudo systemctl stop NetworkManager
sudo ip addr flush dev eth0
sudo ip link set eth0 up
sudo ip addr add 10.0.0.4/24 dev eth0
sudo ip route add default via 10.0.0.1
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

### Verification

```bash
ip a show eth0          # confirm the IP is assigned
ip route                # confirm default route via 10.0.0.1
ping -c 4 10.0.0.1      # gateway reachable
ping -c 4 8.8.8.8       # internet reachable
ping -c 4 google.com    # DNS working
```

## Troubleshooting Log

The lab worked on day one. On day two the VM had no internet. This was the most valuable part of the project.

**Symptom:** `eth0` was UP with the cable connected, but `ip a` showed no `inet` address.

**Errors encountered:**

- `dhclient: command not found` (no DHCP client installed)
- `Error: Connection activation failed: IP configuration could not be reserved (no available address, timeout, etc.)` from `nmcli`

**What I tried, in order:**

1. Checked the link state with `ip a` and found the interface was up but had no IP.
2. Tried `dhclient`, then `nmcli device connect eth0`, which failed with the error above.
3. Created a static NetworkManager profile with the IP my instructor provided (10.0.0.2). It still failed.
4. Bypassed NetworkManager entirely, stopped the service, and configured the interface by hand with the `ip` command, using 10.0.0.4.
5. Tested in layers: gateway, then public IP, then DNS.

**Result:** the gateway (10.0.0.1) and 8.8.8.8 both replied, so the VirtualBox NAT Network was healthy. The problem was address assignment, not the network itself.

**Likely root causes:**

- DHCP not being served on the NAT Network after a VirtualBox/host change.
- The assigned address (10.0.0.2) conflicting with, or being reserved for, another service on the NAT Network.

**Key lessons:**

- Test connectivity in layers (link, IP, gateway, internet, DNS) to find exactly where it breaks.
- A "cable connected" interface does not mean it has an address.
- When a management tool (NetworkManager) fails, configuring the interface manually with `ip` helps isolate the fault.
- Static addresses must be unique per VM on the network.

## Commands Reference

| Task | Command |
|---|---|
| Show interfaces and IPs | `ip a` |
| Show routing table | `ip route` |
| Test connectivity | `ping -c 4 <target>` |
| NetworkManager status | `nmcli device status` |
| Restart NetworkManager | `sudo systemctl restart NetworkManager` |
| Flush an interface's IPs | `sudo ip addr flush dev eth0` |

## Safety and Ethics

- All testing is done only against machines I own or have permission to test inside this lab.
- The lab is isolated through the NAT Network so attack traffic cannot reach external devices.
- Nothing here should be used against systems without explicit authorization.

## Skills Demonstrated

- Virtualization (VirtualBox)
- Linux administration (Kali)
- Network configuration: IP addressing, subnets, gateways, DNS
- Network troubleshooting with `ip`, `ping`, `nmcli`, and routing tables
- Documentation

## Next Steps

- [ ] Add a vulnerable target VM (Metasploitable 2 or DVWA)
- [ ] Practice network scanning with Nmap
- [ ] Explore vulnerability scanning and basic exploitation
- [ ] Make the static IP configuration persistent across reboots
- [ ] Enable DHCP on the NAT Network and compare

## Screenshots


```
![Network settings]<img width="596" height="392" alt="network settings" src="https://github.com/user-attachments/assets/9da1660a-b7f9-432c-9bc0-0eb4535dbdd3" />

![Successful ping]<img width="1536" height="864" alt="successful ping" src="https://github.com/user-attachments/assets/dc30fdc6-1c1d-4744-81c9-06e2acb99ea6" />

```

## Acknowledgments

Thanks to my instructor Waqas Karim for the guidance and lab assignment.

## Author

Alabi Abdulhameed Ayomikun | [LinkedIn URL]
