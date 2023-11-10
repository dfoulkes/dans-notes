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
<warning>This is only for Ubuntu Desktop NOT server</warning>
<step>ssh onto the VM</step>
<step>TODO</step>
</procedure>
