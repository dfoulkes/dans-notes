# Post BIOS Update

<tip>
This document came about due to the multiple issues I had with restoring my Arch after upgrading my BIOS.
</tip>

## Overview

### Index of Issues
 - Ethernet card change it's ID causing the networkctl to be unable to start.
 - Windows Decided to update the EFI boot entry, causing the systemd-boot to be ignored 
 - Secure Boot keys were lost, preventing Arch Linux from booting.

## Fixes


<procedure title="Ethernet Card ID Change Fix">
<tip> The most obvious issue here is you won't have an internet / network connection.</tip>
<step number="1" title="Check networkctl">
Check the status of the network interfaces using:
<code-block language="bash">
    networkctl list
 </code-block> 
Which might look something like this:
<code-block language="bash">
IDX LINK     TYPE     OPERATIONAL SETUP     
  1 lo       loopback carrier     unmanaged
  2 eno1     ether    off         unmanaged
  4 wlp101s0 wlan     off         unmanaged
  5 enp99s0  ether    degraded    unmanaged
</code-block>
<tip> the clue here, is the `degraded`</tip> 
Capture the name of the device that is degraded, in this case `enp99s0`.
</step>
<step number="2" title="verify it has the same IDX as in /etc/systemd/network/20-wired.network">
open in an editor:
<path>
    /etc/systemd/network/20-wired.network
</path> this should look something like:   
<code-block language="ini">
[Match]
# Name=enp10s0
Name=enp99s0

[Network]
DHCP=yes
</code-block>

if `Name=` is different from the IDX captured in step 1, update it to match.
</step>
<step number="3" title="Restart systemd-networkd">
Restart the network service to apply the changes:
<code-block language="bash">
    sudo systemctl restart systemd-networkd
</code-block>
</step>
</procedure>



## Related Incident Topics
- [Guide to setting up secureboot](Secureboot.md)
- [Asus XG-C100C v2 Rev 3](Asus-XG-C100C-v2-Rev-3.md)
- [Booting Issues with Samsung T7](Booting-Issues-with-Samsung-T7.md)