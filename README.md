# Dual-Boot Bluetooth Headphone Auto-Connect (Windows + Linux)

Dual-booting Windows and Linux causes and issue with Bluetooth devices, causing them to lose their pairing between systems. This means you have to remove and re-pair the device each time you switch operating systems.

This guide aims to fix that issue by making your Bluetooth device connect automatically in both systems, without needing to re-pair every time you reboot.

This guide will **synchronize the trust and key info** between systems.

Tested on:
- **Windows 11 24H2**
- **Linux Mint 22.1**

This should work on most linux distributions 
  
## Step 1: Extract the Bluetooth Pairing Key from Windows

To allow Bluetooth devices to connect seamlessly across dual boot, we need to copy the pairing key (called the **link key**) from Windows to Linux. This allows Linux to recognize and trust the device without needing to re-pair.

---

### 1.1 Pair the Bluetooth Headphones in Windows

First, make sure your headphones are **fully paired and connected** in Windows.

1. Open **Settings** → **Bluetooth & devices**
2. Click **Add device** → Select **Bluetooth**
3. Put your headphones in pairing mode, select them, and wait until they show as **connected**

After pairing, Windows stores the device’s encryption key (called the “link key”) in the registry.

---

### 1.2 Download PsExec (Required to Read the Key)

To view the Bluetooth keys, you need to run `regedit.exe` as the **SYSTEM** user.

Download **PsExec** from Microsoft’s official Sysinternals page:  
https://learn.microsoft.com/en-us/sysinternals/downloads/psexec

Extract the ZIP and place `PsExec.exe` somewhere accessible (e.g. `C:\Tools\`).

---
### 1.5 Find Your Device's MAC Address

To identify which folder belongs to your Bluetooth device, you need its MAC address.

1. Press `Win + X` → open **Terminal (Admin)** or **Command Prompt (Admin)**
2. Run this command:

```
getmac /v /fo list
```

Alternatively, for more detailed Bluetooth info, run:

```
Get-PnpDevice -Class Bluetooth | Select-Object Name,InstanceId
```

Look for your device's name (e.g. "Bluetooth mouse", "Sony WH-1000XM5", etc.).  
Its MAC address will appear at the end of the `InstanceId`.

---

### 1.3 Launch Regedit as SYSTEM

Open **Command Prompt as Administrator**, then run:

```
C:\Tools\PsExec.exe -i -s regedit.exe
```

This opens the Registry Editor as the SYSTEM user.

---
### 1.4 Locate the Key

In Registry Editor, navigate to (You can copy paste into the address bar at the top of the Registry Editor):
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\BTHPORT\Parameters\Keys
```

