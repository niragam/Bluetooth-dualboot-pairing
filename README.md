# Persistent Bluetooth Pairing Across Windows & Linux Dual Boot Guide

Dual-booting Windows and Linux often causes Bluetooth devices to lose their pairing, forcing you to remove and re-pair them each time you switch operating systems.

This guide will help you make your Bluetooth device connect automatically on both systems, without needing to re-pair after each reboot.


Tested on:
- **Windows 11 24H2**
- **Linux Mint 22.1**
- **Bose QC Ultra Bluetooth Headphones**

This guide should work for most Linux distributions that use the standard BlueZ Bluetooth stack—including Ubuntu, Debian, Mint, Fedora, Arch, Manjaro, openSUSE, and others.
If your Linux distribution uses a highly customized or non-standard Bluetooth stack, some paths or procedures may differ.

## Before You Start
Pair the device once in Linux first. (This creates the device folder and info file BlueZ needs)
  
## Step 1: Retrieve Bluetooth Keys from Windows

To allow Bluetooth devices to connect seamlessly across dual boot, we need to copy the **link key** from Windows to Linux. This allows Linux to recognize and trust the device without needing to re-pair.

---

### 1.1 Pair the Bluetooth Headphones in Windows

First, make sure your headphones are **fully paired and connected** in Windows.

1. Open **Settings** → **Bluetooth & devices**
2. Click **Add device** → Select **Bluetooth**
3. Enable pairing mode on your device, select it from the list, and wait until it displays as connected

After pairing, Windows stores the device’s encryption key (called the “link key”) in the registry.

---

### 1.2 Download PsExec (Required to Read the Key)

To view the Bluetooth keys, you need to run `regedit.exe` as the **SYSTEM** user.

Download **PsExec** from Microsoft’s official Sysinternals page:  
https://learn.microsoft.com/en-us/sysinternals/downloads/psexec

Extract the ZIP and place `PsExec.exe` in an accessible location, such as your desktop.

---
### 1.3 Find Your Adapter and Device MAC Addresses

To extract the Bluetooth pairing key, you need:

- The **MAC address of your Bluetooth adapter** 
- The **MAC address of your Bluetooth device** 

#### A. Get the Adapter MAC

1. Press `Win + X` and select **Terminal (Admin)**
2. Run this command:
```
getmac /v /fo list
```
Locate the Bluetooth adapter entry, which will appear similar to:
```
Connection Name:  Bluetooth Network Connection
Network Adapter:  Bluetooth Device (Personal Area Network)
Physical Address: 12-34-56-78-9A-BC
```
Record the Physical Address somewhere accessible such as a txt file(in this example: 12-34-56-78-9A-BC) and format it as:
Bluetooth adapter MAC address: 12:34:56:78:9A:BC


#### B. Get the Device MAC address
In the same Terminal window, run:
```
Get-PnpDevice -Class Bluetooth | Select-Object Name,InstanceId
```
Locate your device by name (e.g., "Bluetooth mouse", "Sony WH-1000XM5").
Example output for a device named "MyBluetoothDevice":

```
Name                             InstanceId
----                             ----------
MyBluetoothDevice                BTHENUM\DEV_112233445566\8&ABCDEF&0&BLUETOOTHDEVICE_112233445566
MyBluetoothDevice Avrcp Transport BTHENUM\{1111110C-1100-1000-8110-11305A4B31BA}_VID&0001AAAA_PID&1234\8&ABCDEF&0&...
LE-MyBluetoothDevice             BTHLE\DEV_112233445566\8&12345678&0&112233445566
```
Select the entry that matches only the device name (the main pairing entry):
```
MyBluetoothDevice                BTHENUM\DEV_112233445566\8&ABCDEF&0&BLUETOOTHDEVICE_112233445566
```
The MAC address appears twice in the InstanceId:
```
DEV_112233445566
BLUETOOTHDEVICE_112233445566
```
Record the device MAC address (in this example: 112233445566) and format it as:
```
Bluetooth Device MAC address: 11:22:33:44:55:66
```
Your record/txt file should now look like this:
```
Bluetooth adapter MAC address: 12:34:56:78:9A:BC
Bluetooth Device MAC address: 11:22:33:44:55:66
```
---

### 1.4 Launch Regedit as SYSTEM

In the Terminal, execute (adjust the path to match your PsExec location):

```
.\desktop\PsExec.exe -i -s regedit.exe
```

This opens the Registry Editor with SYSTEM user privileges.

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
Its data will display in hexadecimal format and look something like this:

```
8a 1f b3 92 4d c7 5a 33 19 ee 84 7b 00 af 61 d2
```

This is the link key we’re after, you’ll need to copy this value for use in Linux.

---

### 1.6 Save Your Bluetooth Keys for Use in Linux

**Before you reboot into Linux,** make sure you’ve securely copied all the necessary information from Windows and saved it somewhere accessible—such as a USB drive, cloud storage, or your phone. You will need the following:

- **Bluetooth adapter MAC address** (e.g., `12:34:56:78:9A:BC`)
- **Bluetooth device MAC address** (e.g., `11:22:33:44:55:66`)
- **Link key** value (the hexadecimal string you copied from the Windows registry, e.g., `8a1fb3924dc75a3319ee847b00af61d2`)
  
---

## Step 2: Configure the Link Key in Linux

You will now insert the Windows link key into Linux's Bluetooth configuration to establish device trust and pairing compatibility.

### 2.1 Boot Into Linux

Start your Linux system

---

### 2.2 Confirm the Bluetooth Adapter Folder

Bluetooth keys in Linux are stored in the directory:
```
sudo ls /var/lib/bluetooth/
```
This should show you one or more folders named after your Bluetooth adapter’s MAC address, the same one as we had on windows.

Example output:
```
12:34:56:78:9A:BC
```
---

### 2.3 Confirm the Device Folder
List devices paired with your adapter (replace with your adapter’s MAC address):

```
sudo ls /var/lib/bluetooth/<INSERT ADAPTER ADDRESS>/
```

For example:
```
sudo ls /var/lib/bluetooth/12:34:56:78:9a:bc/
```
his should show you a folder named after your Bluetooth devices’s MAC address, the same one as we had on windows, and other folders.

Example output:
```
11:22:33:44:55:66  cache  settings
```

---

### 2.4 Edit the Device’s Info File

Open the device's configuration file using a text editor:
```
sudo nano /var/lib/bluetooth/<INSERT ADAPTER ADDRESS>/<INSERT DEVICE ADDRESS>/info
```
your command should look something like:
```
sudo nano /var/lib/bluetooth/12:34:56:78:9a:bc/11:22:33:44:55:66/info
```

---

### 2.5 Insert the Windows Link Key
Inside the info file, locate (or create) a section like this:

```
[LinkKey]
Key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Type=4
PINLength=0
```
Replace the x characters with your Windows link key (use lowercase letters, remove all spaces).
If the [LinkKey] section does not exist, manually add it to the file.

Save with Ctrl+O, press enter to confirm then exit with Ctrl+X.

---

### 2.6 Configure File Permissions (Optional)
Normally Linux sets the correct permissions automatically.
If your device still won’t connect, make sure the file is owned by root and only readable by root:
```
sudo chown root:root /var/lib/bluetooth/<INSERT ADAPTER ADDRESS>/<INSERT DEVICE ADDRESS>/info
sudo chmod 600 /var/lib/bluetooth/<INSERT ADAPTER ADDRESS>/<INSERT DEVICE ADDRESS>/info
```

---

### 2.7 Restart Bluetooth
Restart the Bluetooth service to apply changes:
```
sudo systemctl restart bluetooth
```

---

### 2.8 Test the Connection
Turn on your Bluetooth device. It should now connect automatically in Linux without needing to be re-paired.

During the initial connection attempt, you may need to manually connect the device **without re-pairing** if the system has not yet restarted.
