# Post BIOS Update

> **Note:** This document came about due to the multiple issues I had with restoring my Arch after upgrading my BIOS.

## Overview

### Index of Issues
 - Ethernet card change it's ID causing the networkctl to be unable to start.
 - Windows Decided to update the EFI boot entry, causing the systemd-boot to be ignored 
 - Secure Boot keys were lost, preventing Arch Linux from booting.

## Fixes

### Ethernet Card ID Change Fix

> **Note:** The most obvious issue here is you won't have an internet / network connection.

#### Step 1: Check networkctl

Check the status of the network interfaces using:

```bash
networkctl list
```

Which might look something like this:

```bash
IDX LINK     TYPE     OPERATIONAL SETUP     
  1 lo       loopback carrier     unmanaged
  2 eno1     ether    off         unmanaged
  4 wlp101s0 wlan     off         unmanaged
  5 enp99s0  ether    degraded    unmanaged
```

> **Note:** The clue here, is the `degraded`

Capture the name of the device that is degraded, in this case `enp99s0`.

#### Step 2: Verify it has the same IDX as in /etc/systemd/network/20-wired.network

Open in an editor: `/etc/systemd/network/20-wired.network`

This should look something like:

```ini
[Match]
# Name=enp10s0
Name=enp99s0

[Network]
DHCP=yes
```

If `Name=` is different from the IDX captured in step 1, update it to match.

#### Step 3: Restart systemd-networkd

Restart the network service to apply the changes:

```bash
sudo systemctl restart systemd-networkd
```



## Related Incident Topics
- [Guide to setting up secureboot](Secureboot.md)
- [Asus XG-C100C v2 Rev 3](Asus-XG-C100C-v2-Rev-3.md)
- [Booting Issues with Samsung T7](Booting-Issues-with-Samsung-T7.md)