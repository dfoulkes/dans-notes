# True NAS Cluster Management


## Getting the status of zpool


```
zpool status


pool: boot-pool
state: ONLINE
scan: scrub repaired 0B in 00:00:02 with 0 errors on Tue Apr  8 03:45:03 2025
config:

        NAME         STATE     READ WRITE CKSUM
        boot-pool    ONLINE       0     0     0
          nvme0n1p3  ONLINE       0     0     0

errors: No known data errors

pool: home_pool
state: DEGRADED
status: One or more devices are faulted in response to persistent errors.
Sufficient replicas exist for the pool to continue functioning in a
degraded state.
action: Replace the faulted device, or use 'zpool clear' to mark the device
repaired.
scan: resilvered 2.99G in 00:00:29 with 0 errors on Tue Apr  8 00:03:39 2025
config:

        NAME                                      STATE     READ WRITE CKSUM
        home_pool                                 DEGRADED     0     0     0
          raidz2-0                                DEGRADED     0     0     0
            5eecca48-960f-41e7-8ada-f7009268da3a  ONLINE       0     0     0
            40055ce3-3420-432b-9340-ed725bcd90f4  FAULTED     14     0     0  too many errors
            b057666d-7581-4bfe-bfdc-c36e01ebc8be  ONLINE       0     0     0
            79f45321-0375-49eb-845b-e4c221e4776c  ONLINE       0     0     0

```

When a drive is `Faulted` it means that the drive is not responding to the system. This can be due to a number of reasons, including:
- The drive has failed
- The drive is not properly connected to the system
- The drive is not powered on

After first checking points 1 and 2, if the drive is still `Faulted`, it is likely that the drive has failed. In this case, you will need to replace the drive.

## Replacing a Faulted Drive
![nas_check_drive_health.png](nas_check_drive_health.png)

![nas_faulted_drive.png](nas_faulted_drive.png)
<procedure title="Replace a Faulted Drive">
<step>
<p>Check the status of the zpool as shown above image in the Truenas UI.
   document the `Faulted` drive Device Name i.e `sdd`.</p>
</step>
<step>
<p>From either `ssh` or via the `System -> Shell` Check the status of the drive using smartctl</p>
<code-block lang="Bash">
sudo smartctl -a /dev/sdd
</code-block>
<code-block lang="Bash">
SMART Self-test log structure revision number 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Extended offline    Completed: read failure       90%        16         16787464
# 2  Extended offline    Completed: read failure       90%         7         15624
# 3  Short offline       Completed without error       00%         6         -
# 4  Short offline       Completed without error       00%         3         -
# 5  Short offline       Aborted by host               90%         3         -
# 6  Short offline       Completed without error       00%         0         -
</code-block>
</step>
<step>
<p>Now that we have the device id, we need to turn off the system.
Once booted down, physically remove the drive from the system.
Replace the drive with a new one and boot the system back up.
</p>
</step>
<step>
<p>Once the system is back up, check the status of the zpool again.</p>
<code-block lang="Bash">
zpool status
</code-block>
<p>
Verify that the `Faulted` drive is no longer listed.
</p>
<p>To replace the old disk in the pool, run the following command.</p>
<code-block lang="Bash">
zpool replace home_pool 40055ce3-3420-432b-9340-ed725bcd90f4 sdd
</code-block>
<p>Where `home_pool` is the name of the pool, `40055ce3-3420-432b-9340-ed725bcd90f4` is the ID of the old disk, and `sdd` is the name of the new disk.</p>
</step>
</procedure>


