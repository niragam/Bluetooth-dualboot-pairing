# Persistent Bluetooth Pairing Across Windows & Linux Dual Boot Guide

Dual-booting Windows and Linux often causes Bluetooth devices to lose their pairing, forcing you to remove and re-pair each time you switch OSes.

This guide aims to fix that issue by making your Bluetooth device connect automatically in both systems, without needing to re-pair every time you reboot.

This guide will **synchronize the trust and key info** between systems.

Tested on:
- **Windows 11 24H2**
- **Linux Mint 22.1**

This should work on most linux distributions 

## Before You Start
Make sure the Bluetooth device has been paired at least once in Linux.
  
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
### 1.3 Find Your Adapter and Device MAC Addresses

To extract the Bluetooth pairing key, you need:

- The **MAC address of your Bluetooth adapter** (your PC's built-in or USB Bluetooth)
- The **MAC address of your Bluetooth device** (e.g. headphones)

#### A. Get the Adapter MAC

1. Press `Win + X` → open **Terminal (Admin)**
2. Run this command:
```
getmac /v /fo list
```
Look for the Bluetooth adapter e.g.:
```
Connection Name:  Bluetooth Network Connection
Network Adapter:  Bluetooth Device (Personal Area Network)
Physical Address: 12-34-56-78-9A-BC
```
Note its **Physical Address** - in this case: `12-34-56-78-9A-BC`


#### B. Get the Device MAC address
Still in the Terminal, run:
```
Get-PnpDevice -Class Bluetooth | Select-Object Name,InstanceId
```
Look for your device's name (e.g. "Bluetooth mouse", "Sony WH-1000XM5", etc.).  

Example output for device called MyBluetoothDevice:

```
Name                             InstanceId
----                             ----------
MyBluetoothDevice                BTHENUM\DEV_112233445566\8&ABCDEF&0&BLUETOOTHDEVICE_112233445566
MyBluetoothDevice Avrcp Transport BTHENUM\{1111110C-1100-1000-8110-11305A4B31BA}_VID&0001AAAA_PID&1234\8&ABCDEF&0&...
LE-MyBluetoothDevice             BTHLE\DEV_112233445566\8&12345678&0&112233445566
```
Choose the entry that matches just the device name, This is the main pairing entry.
```
MyBluetoothDevice                BTHENUM\DEV_112233445566\8&ABCDEF&0&BLUETOOTHDEVICE_112233445566
```
You'll see the MAC address in two places in each InstanceId:
```
DEV_112233445566
BLUETOOTHDEVICE_112233445566
```
Note the device MAC address, In this case, the MAC address is `11:22:33:44:55:66`.


---

### 1.4 Launch Regedit as SYSTEM

In terminal run (change according to PsExec location):

```
C:\Tools\PsExec.exe -i -s regedit.exe
```

This opens the Registry Editor as the SYSTEM user.

---
### 1.5 Locate the Link key

In Registry Editor, navigate to (You can copy paste into the address bar at the top of the Registry Editor):
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\BTHPORT\Parameters\Keys
```
Find the folder that matches your adapter’s physical address.
For example, if your adapter MAC is `12-34-56-78-9A-BC`, the folder name will be:
```
123456789abc
```

Inside that folder you will see a `REG_BINARY`  value whose name matches your Bluetooth device’s MAC address.
For example For `11:22:33:44:55:66` will appear as:
```
112233445566
```
Its data will look something like this:

```
8a 1f b3 92 4d c7 5a 33 19 ee 84 7b 00 af 61 d2
```

This is the link key we’re after, you’ll need to copy this value for use in Linux.


