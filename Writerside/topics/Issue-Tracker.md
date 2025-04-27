# Issue Tracker



## Issue Log

<deflist collapsible="true">
    <def title="Failed to allocate directory watch: Too many open files" default-state="expanded">
    <br />
    <p>Noticing issues on master1a, seeing this error message when trying to install tools.</p>
    <code lang="bash">
         Failed to allocate directory watch: Too many open files
    </code>
    <p> Also whilst inspecting the <code>dmesg</code>  </p>
    <note type="info">
    This is also the node in which loki runs as it has the M2 drive. For note we're seeing
    errors in alertmanager of:
    <code-block lang="bash">
     2025-04-21 09:52:49.457 failed to create fsnotify watcher: too many open files
     </code-block>
   </note>
   <code-block lang="bash">
    [1039859.270706] EXT4-fs error (device sdc) in ext4_reserve_inode_write:5812: Journal has aborted
    [1039859.270721] EXT4-fs error (device sdc) in ext4_reserve_inode_write:5812: Journal has aborted
    [1039859.270725] EXT4-fs error (device sdc): ext4_handle_inode_extension:319: inode #1449868: comm minio: mark_inode_dirty error
    [1039859.270728] EXT4-fs error (device sdc) in ext4_handle_inode_extension:321: Journal has aborted
    [1039859.270731] EXT4-fs error (device sdc): ext4_journal_check_start:84: comm minio: Detected aborted journal
    [1039859.327640] sd 2:0:0:1: [sdc] tag#84 Sense Key : 0x3 [current]
    [1039859.327644] sd 2:0:0:1: [sdc] tag#84 ASC=0x11 ASCQ=0x0
    [1039859.327646] sd 2:0:0:1: [sdc] tag#84 CDB: opcode=0x28 28 00 00 4b d2 80 00 00 20 00
    [1039864.032370] EXT4-fs (sdc): Remounting filesystem read-only
    [1039864.032480] EXT4-fs (sdc): Remounting filesystem read-only
    </code-block>
    <p> This is a result of the filesystem being in a read-only state. This can happen if the system detects an error with the filesystem, 
       such as a corrupted journal or a hardware issue with the disk.</p>
    <p> After a sereis of googling and ChatGPT questions, the next thing I will try is</p>
    <code-block lang="bash">
    sudo sysctl -w fs.inotify.max_user_watches=262144
    sudo sysctl -w fs.inotify.max_user_instances=256 # Double it for good measure
    </code-block>
    <p>we now wait, see if the error messages in alert manger drop off.</p>
    <p>Update 12/04 - This seemed to have fixed the error. </p>
    </def>
    <def title="Collapsed by default" default-state="collapsed">
        This is the definition of the second term.
    </def>
</deflist>