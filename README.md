# Intercepting HTTP/HTTPS Traffic on iOS Using OpenVPN (Kali) + SSL Pinning Bypass

This guide explains how to intercept iOS application traffic when traditional proxy methods fail.  
Since `iptables` cannot be used directly on iOS, we configure a **VPN server on Kali Linux** and route device traffic through it.

This method is commonly used during **iOS Mobile VAPT**.

---

## ⚠️ Disclaimer

This guide is intended strictly for **educational and authorized security testing purposes only**.  
Test only devices and applications you own or have explicit permission to assess.

---

# 📡 Part 1 — Setting Up OpenVPN on Kali

## Step 1 — Download and Install OpenVPN Script

Run the following on Kali:

```bash
wget https://git.io/vpn -O openvpn-install.sh
sed -i "$(($(grep -ni "debian is too old" openvpn-install.sh | cut -d : -f 1)+1))d" ./openvpn-install.sh
chmod +x openvpn-install.sh
sudo ./openvpn-install.sh
```

---

## Step 2 — Configuration Options

During setup, choose:

- **IPv4 address** → Your Kali local IP address  
- **Behind NAT?** → Enter your local IP again  
- **Protocol** → `1` (UDP)  
- **Port** → `1194`  
- **DNS server** → Choose preferred (e.g., `1.1.1.1`)  
- **Client name** → Choose any name  

---

## Step 3 — Verify VPN Interface

Run:

```bash
ifconfig
```

You should see a new interface:

```
tun0
```

---

## Step 4 — Start OpenVPN Service

```bash
sudo service openvpn start
```

---

# 📲 Part 2 — Install OpenVPN Profile on iOS

## Step 1 — Host the .ovpn File

By default, the client file is stored in `/root`.

Start a Python HTTP server:

```bash
sudo python3 -m http.server 8080 --directory /root/
```

---

## Step 2 — Download on iPhone

On iPhone browser, visit:

```
http://<KaliLocalIP>:8080
```

Download the `.ovpn` file.

---

## Step 3 — Import to OpenVPN App

1. Install **OpenVPN Connect** from App Store  
2. Open the downloaded `.ovpn` file  
3. Import configuration  
4. Connect to VPN  

---

# 🔀 Part 3 — Route Traffic to Burp Suite

Since traffic is now routed through Kali via VPN, we redirect it using `iptables`.

---

## Step 1 — Configure NAT Redirection

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination <BurpsuiteIP>
```

---

## Step 2 — Configure Burp

1. Open **Burp Suite**
2. Proxy → Proxy Settings
3. Listen on port `443`
4. Enable **Invisible Proxy Mode**

Now HTTPS traffic from iOS flows through Kali and into Burp.

---

# 🔐 Part 4 — iOS SSL Pinning Bypass

Even after interception is set up, many applications implement **SSL Pinning**, preventing traffic inspection.

---

## What is SSL Pinning?

SSL Pinning ensures the app only trusts a specific certificate or public key.  
If Burp’s certificate is presented, the connection is blocked.

---

## SSL Pinning Bypass Methods

### 1️⃣ Jailbroken Device + Frida (Recommended for Testing)

Install Frida and use a generic iOS SSL bypass script:

```bash
frida -U -n <AppName> -l ios_ssl_bypass.js
```

This hooks SSL validation functions at runtime.

---

### 2️⃣ Objection

```bash
objection -g com.app.package explore
ios sslpinning disable
```

---

### 3️⃣ Binary Patching (Advanced)

- Dump IPA
- Analyze using Hopper / IDA
- Patch SSL validation functions
- Re-sign IPA

---

## Common iOS SSL Pinning Targets

- `SecTrustEvaluate`
- `NSURLSession`
- `AFNetworking`
- `TrustKit`

---

# 🔥 Common Issues

### Traffic Not Visible?
- Ensure VPN is connected
- Check tun0 interface
- Verify iptables rules
- Confirm Burp is listening on correct port

---

### App Still Blocking?
- SSL pinning likely implemented
- Use Frida or dynamic instrumentation

for frida
connect phone to kali using ssh
```
ssh mobile@ip
cd /usr/sbin/frida-server -l <ip of mobile>
open new terminal in kali
frida-ps -aH <ip of mobile>
```
- Runtime analysis
- SSL Pinning assessment

---

## ✅ End of Guide
