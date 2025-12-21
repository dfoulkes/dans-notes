# Arch Backup Image

## Overview
This document covers the steps that I took in creating a 
backup image of the T7 which hosts the Arch Linux installation.



## Backup Procedure

<procedure title="Backup T7 from Windows WSL2">
<step number="1" title="mounting the T7 in WSL2">
First, ensure that the Samsung T7 is connected to the Windows host machine.
from an admin PowerShell window run the following:
<code-block language="powershell">
wsl --mount \\.\PHYSICALDRIVE3 --bare
</code-block>
<note>
Replace `PHYSICALDRIVE3` with the appropriate drive number for your T7.
</note>
</step>
<step number="2" title="Identify the device in WSL2">
Once mounted, open your WSL2 terminal.
<code-block language="bash">
lsblk
</code-block>

which should look something like this:
<code-block language="bash">
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 388.4M  1 disk
sdb      8:16   0   186M  1 disk
sdc      8:32   0     8G  0 disk [SWAP]
sdd      8:48   0 931.5G  0 disk
├─sdd1   8:49   0   512M  0 part
├─sdd2   8:50   0   890G  0 part
└─sdd3   8:51   0    41G  0 part
sde      8:64   0     1T  0 disk /snap
                                 /mnt/wslg/distro
                                 /
</code-block> 
In this example we know it will be `sdd` based on the size of the drive,
the partitions which we recognise as the EFI, root and home partitions.
</step>
<step number="3" title="Creating the backup image">
Now we can create the backup image using `dd`.
<code-block language="bash">
sudo dd if=/dev/sdd bs=64M status=progress | zstd -T0 -1 | dd of=/mnt/nas/arch_t7_backup.img.zst bs=4M
</code-block>

- Replace `/dev/sdd` with the correct device identifier for your T7.
- Replace `/mnt/nas/arch_t7_backup.img.zst` with the desired path and filename for the backup image.
- The image is compressed using `zstd` to save space.
<note>
In this example, the end `zst` compressed image will be stored on the NAS.
</note>
</step>
</procedure>

## Restore Procedure


<procedure>
<step number="1" title="Mounting the T7 in WSL2">
Remount the Samsung T7 in WSL2 from an admin PowerShell window:
<code-block language="powershell">
wsl --mount \\.\PHYSICALDRIVE3 --bare
</code-block>
</step>
<step number="2" title="Identify the device in WSL2">
Once mounted, open your WSL2 terminal.
<code-block language="bash">
lsblk
</code-block>

which should look something like this:
<code-block language="bash">
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 388.4M  1 disk
sdb      8:16   0   186M  1 disk
sdc      8:32   0     8G  0 disk [SWAP]
sdd      8:48   0 931.5G  0 disk
├─sdd1   8:49   0   512M  0 part
├─sdd2   8:50   0   890G  0 part
└─sdd3   8:51   0    41G  0 part
sde      8:64   0     1T  0 disk /snap
                                 /mnt/wslg/distro
                                 /
</code-block> 
Identify the correct device for your T7, in this example it is `sdd`.
</step>
<step number="3" title="Restoring the backup image">
To restore the backup image to the T7, use the following command:
<code-block language="bash">
zstd -d -c /mnt/nas/arch_t7_backup.img.zst | sudo dd of=/dev/sdX bs=64M status=progress
</code-block> 
This uses the zstd decompression to read the backup image and pipes it to `dd` to write it back to the T7.

<note>
Replace `sdX` with the correct device identifier for your T7.
</note>
</step>
</procedure>