# Learning Network Security the Fun Way

Just messing around with network stuff to understand how attacks actually work. Built a small lab at home and tried some things I kept hearing about.

## ⚠️ Quick Note

This is just me learning. Only did this on my own network with my own devices. Don't be stupid and test this on networks you don't own - that's illegal pretty much everywhere.

---

## What I Tried

### Deauth Attacks (Kicking Devices Off WiFi)

**The idea:** WiFi has these "management frames" that tell devices to disconnect. Turns out anyone can send these, even if you're not connected to the network.

**What happened:**
```bash
# Put wifi card in monitor mode
sudo airmon-ng start wlan0

# See what networks are around
sudo airodump-ng wlan0mon

# Send disconnect packets
sudo aireplay-ng --deauth 10 -a <router_mac> wlan0mon
```

My phone got kicked off instantly. Reconnected after a few seconds, but kept getting kicked while the command was running.

**Why it works:**
- Old WiFi (WPA2) doesn't check if disconnect messages are real
- Your device just trusts them and disconnects
- It's basically WiFi's version of "Simon says disconnect" and devices have to listen

**How to stop it:**
- Use WPA3 (encrypts management frames)
- Or enable 802.11w/PMF on WPA2 routers
- Most new routers have this, you just need to turn it on

**What I learned:** This is stupidly easy. Like, scary easy. Also explains why coffee shop WiFi keeps dropping sometimes.

---

### ARP Poisoning (MITM / Sitting in the Middle)

**The idea:** Your computer asks "who has this IP?" and trusts whatever response it gets. You can lie and say "that's me!" and intercept traffic.

**What happened:**
```bash
# Let my computer forward packets
sudo sysctl net.ipv4.ip_forward=1

# Tell the phone I'm the router, tell the router I'm the phone
sudo ettercap -T -M arp:remote /router_ip// /phone_ip// -i eth0

# Watch what goes through
sudo tcpdump -i eth0 host phone_ip
```

I could see every website my phone visited (the DNS requests), what apps were connecting to what servers, basically all the traffic flowing through.

**What I could see:**
- DNS queries (every site visited)
- HTTP traffic (headers, content - but nobody uses HTTP anymore)
- Connection patterns (what apps talking to what servers)
- Timings (when stuff was accessed)

**What I couldn't see:**
- HTTPS content (encrypted, just saw metadata)
- VPN traffic (totally encrypted)
- App data (most apps use encryption now)

**How to stop it:**
- Use a VPN (encrypts everything before it leaves your device)
- HTTPS everywhere (most sites do this now anyway)
- For networks you control: enable ARP inspection on your switch
- Check your ARP table occasionally: `arp -a`

**What I learned:** 
- This attack is weirdly effective even though everything's "encrypted" now
- You can't see content, but you can see A LOT about behavior
- Also learned: my phone talks to SO MANY servers constantly. Like, constantly.

---

## Tools I Used

**For testing:**
- `aircrack-ng` suite (WiFi stuff)
- `ettercap` (ARP poisoning)
- `wireshark` (looking at packets)
- `nmap` (finding devices)

**For detection/defense:**
- `arpwatch` (alerts on ARP changes)
- `tcpdump` (quick packet checking)
- Built a simple Python script to watch for weird ARP behavior

---

## My Setup

- Old laptop running Pop OS (Ubuntu distro)
- Builtin WiFi adapter
- Old Android phone
- test network

---

## Stuff That Would Be Cool to Try Next

- Building a rogue AP detector
- Making a dashboard that shows attacks in real-time
- Testing WPA3 to see if it actually stops this stuff
- Trying to detect these attacks automatically
- Building a honeypot network to see what attackers do

---

## Random Things I Learned

- WiFi security is weirdly broken in old standards
- Most "hacking" is just understanding protocols and using existing tools
- Defense is actually harder than offense
- Encryption solves most problems, but metadata leaks are real
- Default router settings are garbage
- WPA3 exists and actually fixes most of this, but nobody uses it yet

---

## Resources I Used

**Learned from:**
- NetworkChuck videos
- David Bombal's channel
- TryHackMe labs
- Random blog posts and forums

**Tools docs:**
- Aircrack-ng wiki
- Ettercap documentation
- Wireshark tutorials

---

**Disclaimer:** This is for learning. Don't test on networks you don't own. Be smart.
