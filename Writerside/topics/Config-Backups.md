# Config Backups

## What is it?

To improve the speed of system recovery, this script backs up the 
winget installed packages export. This can be used to restore the
systems software in a disaster recovery scenario.


## Setup 


### Installing NasBackup Util
The config is backed up onto TrueNas here:
<note>\\TRUENAS\home-pool\dfoulkes\winget-backup</note>

In the event of a disaster, the following steps should be taken:
<procedure title="Restore Config">
   <p>Ensure Winget is installed on the new system.</p>
   <step>Check Winget is available in powershell with <code>winget --version</code></step>
</procedure>

<procedure title="Reinstalling the NasBackup Util">
   <p>On a fresh install of Windows, reinstall the `nasbackup util` </p>
   <warning>For this, you will at least need Python 3.12.0 installed first. and git</warning>
   <step>clone <a href="https://github.com/dfoulkes/BackupConfigtoNas">dfoulkes/BackupConfigtoNas</a> </step>
   <step>
   To build and install, run the following command:
   <code>python setup.py install</code>
   </step>
   <step>verify the install with <code>Get-Command nasbackup</code></step>
</procedure>

### Restoring the Config
<procedure title="Reinstall Using the Config file">
  <p>Map the NAS entry</p>
   <img src="winget_restore_map_network_drive.png" alt=""/>
   <step>In explorer, click on <code>...</code> 'Add a network folder'</step>
   <step>Enter the NAS address <code>\\TRUENAS\home-pool\dfoulkes\winget-backup</code> and select 
         a driver letter, for this example set to N:</step>
      <step>Click on <code>Finish</code></step>
    <step>Open Powershell as Administrator</step>
    <step>Run the following command: <code>winget import -i X:\winget-backup\app.json</code></step>
    <step>Wait for the packages to install</step>
</procedure>

### Configure Backup Schedule

## Schedule
<note>Machine needs to be on to run.</note>
The backup script will run every Sunday at 09:00. The job runs a command called `winget-backup` which is compiled python
script. This script is located in the `C:\Users\danfo\AppData\Local\Programs\Python\Python311\Scripts\nasbackup.exe`

## Source Code
The source code for this backup can be found on
[Github dfoulkes/BackupConfigtoNas](https://github.com/dfoulkes/BackupConfigtoNas)

