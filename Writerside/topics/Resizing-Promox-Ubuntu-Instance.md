# Resizing Proxmox Ubuntu Instance


## Updating the Disk Size of VM in Proxmox


<procedure title="Increase Disk Space on Proxmox">
<img src="proxmox_resize_disk_select_vm.png" alt=""/>
   <step>Select the VM to resize</step>
   <step>Click on the Hardware tab</step>
   <step>Click on the Hard Disk</step>
<img src="proxmox_resize_disk_edit_disk.png" alt=""/>
    <step>Click on the Resize Disk button</step>
    <step>Select how much (GB) you want to increase the disk space by.</step>
<step>Click on the Resize Disk button</step>
<step>reboot the VM</step>
</procedure>


<procedure title="Expand Disk Space on Ubuntu Desktop">
<warning>This is only for Ubuntu Desktop NOT server</warning>
<step>ssh onto the VM</step>
<step>
List the current disks
<code xml:lang="bash">
sudo fdisk -l /dev/sda
</code>
</step>
<step>Identify the root partition, for example <tip>Look for the line with: Linux filesystem</tip></step>
<step>In this case the root partition was /dev/sda3, now to resize run <code>sudo growpart /dev/sda 3</code></step>
<step>Now reboot the VM</step>
<step>Once restarted, check the size of the disk with <code>df -h</code></step>
</procedure>


<procedure title="Expand Disk Space on Ubuntu Server">
<warning>
This is ONLY for VMs using LVM, if you're using a standard partition setup, then you can skip the LVM steps and just use growpart and resize2fs to resize the partition and filesystem respectively.
</warning>
<step>ssh onto the VM</step>
<step>
Identify the root partition, and the partition number.
<code xml:lang="bash">
superman@homeassistant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0   74M  1 loop /snap/core22/2339
loop2                       7:2    0 63.8M  1 loop /snap/core20/2686
loop3                       7:3    0   74M  1 loop /snap/core22/2292
loop4                       7:4    0   71M  1 loop /snap/prometheus/86
loop5                       7:5    0 50.9M  1 loop /snap/snapd/25577
loop6                       7:6    0 48.1M  1 loop /snap/snapd/25935
loop7                       7:7    0 63.8M  1 loop /snap/core20/2717
sda                         8:0    0   45G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   28G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   28G  0 lvm  /
</code>
</step>
<step>
<code xml:lang="bash">
sudo apt update
sudo apt install cloud-guest-utils
</code>
</step>
<step>
Use growpart to resize the partition, in this case it's partition 3 on /dev/sda

<code xml:lang="bash">
sudo growpart /dev/sda 3
</code>
</step>
<step>
now resize the physical volume with pvresize.

<code xml:lang="bash">
sudo pvresize /dev/sda3
</code>
</step>
<step>
Now resize the logical volume, in this case it's called ubuntu-lv
<code xml:lang="bash">
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
</code>
</step>
<step>
Now resize the filesystem, in this case it's ext4
<code xml:lang="bash">
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
</code>
</step>
</procedure>
