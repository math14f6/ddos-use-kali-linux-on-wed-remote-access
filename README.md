# ddos-use-kali-linux-on-wed-remote-access
webgui ddos remote use kali linux 
# DDoS Testing Tool

## Overview
This tool is designed for **network stress testing**. It allows you to test the resilience of your server by simulating a DDoS attack. This tool is intended **only for educational purposes** and **testing servers that you own** or have explicit permission to test.

The application is a **web-based interface**, allowing you to control and configure stress tests from your browser.

### Features:
- **Web-based interface** for easy access and control.
- Allows configuration of **target IP**, **ports**, and **attack duration**.
- Utilizes **all available CPU cores** for maximum stress testing.
- **Real-time control** over the attack via the web interface.

## Disclaimer
This tool is provided **"as is"** without any warranty.  
By using this tool, you agree to:
- Only use it to test servers that you own or have **explicit permission** to test.
- Acknowledge that **illegal usage is prohibited** and will result in the immediate revocation of your permission to use this tool.

The author is **not responsible** for any misuse of this tool. Use at your own risk.

## Requirements

### On Kali Linux
To use the tool on **Kali Linux**, you will need to install the following dependencies:
1. **Python 3.x**:
   - Kali Linux usually comes with Python 3.x pre-installed. To ensure you have the latest version, use the following command:
     ```bash
     sudo apt update
     sudo apt install python3
     ```

2. **Pip (Python package installer)**:
   - To install necessary Python libraries, you will need pip:
     ```bash
     sudo apt install python3-pip
     ```

3. **Flask**:
   - This application uses Flask for the web interface. Install Flask using:
     ```bash
     pip3 install Flask
     ```

4. **Hping3**:
   - Hping3 is used to simulate the DDoS traffic. Install Hping3 on Kali Linux by running:
     ```bash
     sudo apt install hping3
     ```

### On General Linux
For general Linux distributions, you can follow the same steps as Kali Linux, with the exception of `apt` being replaced by your distribution's package manager (e.g., `yum`, `dnf`, `pacman`).

- Install **Python 3.x**, **Pip**, **Flask**, and **Hping3** as described above.

### Additional Setup for Remote Access
If you want to access the web interface from other devices:
1. Ensure your firewall allows traffic on port 5000.
2. Run the tool with:
   ```bash
   python3 app.py
