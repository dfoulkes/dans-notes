# Secure Drive Setup


## Securing Arch Linux on External USB with LUKS & TPM2

This guide covers the steps to take to configure a encrypted root drive in Arch Linux using LUKS and TPM2.

WHY? Being a remote storage device it could be very prone to theft or loss, so encrypting the drive ensures that even if the physical device is compromised, the data remains secure.
adding the TPM2 auto-unlock feature enhances usability by allowing the system to boot without manual password entry when connected to the enrolled TPM2 chip with Secure Boot enabled.

## Prerequisites
- Existing Arch Linux installation on an external USB drive (e.g., Samsung T7).
- Arch Linux Live ISO for recovery and setup.
- Basic familiarity with LUKS encryption, systemd-boot, and TPM2.


## Backup & Encryption

<procedure title="Phase 1: Backup and Encryption" id="phase-1-encryption">
    <warning>
    This script makes assumptions that your drive only has two partitions: an EFI partition (e.g., /dev/sdb1) and a root partition (e.g., /dev/sdb2). If the drive has multiple partitions for example 
a dedicated SWAP partition, it's recommended that you flatten the drive to have only two partitions before proceeding. This document does not
    cover multi-partition setups. 
</warning>
    <step>
        <p>Boot into the Arch Linux Live ISO.</p>
    </step>
    <step>
        <p>Create a backup of the existing system using <code>rsync</code> to a separate drive.</p>
        <note>
            <p>Mount your backup destination drive first (e.g., <code>mount /dev/sdX /mnt/backup</code>).</p>
        </note>
        <code-block lang="bash">
            sudo rsync -aAXHvx --info=progress2 \
              --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} \
              / /mnt/backup/arch_backup/
        </code-block>
        <p><b>Flag Explanations:</b></p>
        <list>
            <li><code>-a</code> (archive mode): Preserves permissions, ownership, timestamps, and recursively copies directories</li>
            <li><code>-A</code>: Preserves Access Control Lists (ACLs)</li>
            <li><code>-X</code>: Preserves extended attributes (xattrs)</li>
            <li><code>-H</code>: Preserves hard links</li>
            <li><code>-v</code>: Verbose output to see what's being copied</li>
            <li><code>-x</code>: Don't cross filesystem boundaries (stays on the source filesystem)</li>
            <li><code>--info=progress2</code>: Shows overall progress with transfer speed and time remaining</li>
        </list>
        <p><b>Exclusions Explained:</b></p>
        <list>
            <li><code>/dev/*</code>, <code>/proc/*</code>, <code>/sys/*</code>: Virtual filesystems managed by the kernel</li>
            <li><code>/tmp/*</code>, <code>/run/*</code>: Temporary runtime files</li>
            <li><code>/mnt/*</code>, <code>/media/*</code>: Mount points (prevents copying the backup destination into itself)</li>
            <li><code>/lost+found</code>: Filesystem recovery directory</li>
        </list>
        <warning>
            <p>This backup can take significant time depending on your system size. Ensure you have enough space on the backup drive.</p>
        </warning>
    </step>
    <step>
        <p>Identify the target partition (Root) and formatted it with LUKS2 encryption.</p>
        <note>
            <p>Ensure you are targeting the correct partition (e.g., <code>/dev/sdb2</code>). Do <b>not</b> format the EFI partition (<code>/dev/sdb1</code>).</p>
        </note>
        <code-block lang="bash">
            cryptsetup luksFormat /dev/sdb2
        </code-block>
    </step>
    <step>
        <p>Open the new encrypted container.</p>
        <code-block lang="bash">
            cryptsetup open /dev/sdb2 cryptroot
        </code-block>
    </step>
    <step>
        <p>Format the mapped device with your filesystem of choice (e.g., ext4) and mount the system.</p>
        <code-block lang="bash">
            mkfs.ext4 /dev/mapper/cryptroot
            mount /dev/mapper/cryptroot /mnt
            mkdir /mnt/boot
            mount /dev/sdb1 /mnt/boot
        </code-block>
    </step>
    <step>
        <p>Restore the system backup to the new encrypted partition.</p>
        <code-block lang="bash">
            sudo rsync -aAXHv --info=progress2 /mnt/backup/arch_backup/ /mnt/
        </code-block>
        <note>
            <p>Ensure the trailing slash on the source path (<code>/mnt/backup/arch_backup/</code>) to copy the contents, not the directory itself. The restore uses fewer flags because we're copying to an empty destination and don't need to worry about crossing filesystems.</p>
        </note>
    </step>
</procedure>

## Boot Configuration

<procedure title="Phase 2: USB Boot Configuration (Critical)" id="phase-2-boot-config">
    <step>
        <p>Enter the system environment.</p>
        <code-block lang="bash">
            arch-chroot /mnt
        </code-block>
    </step>
    <step>
        <p><b>Fix 1: Ensure USB drivers are present.</b></p>
        <p>Edit <code>/etc/mkinitcpio.conf</code>. In the <code>HOOKS</code> array, remove <code>autodetect</code> and ensure <code>sd-encrypt</code> is present.</p>
        <p><i>Removing autodetect prevents the installer from stripping USB drivers required to find the external drive at boot.</i></p>
        <code-block lang="bash">
            # /etc/mkinitcpio.conf
            HOOKS=(base systemd modconf keyboard block sd-encrypt filesystems fsck)
        </code-block>
    </step>
    <step>
        <p>Regenerate the initramfs images.</p>
        <code-block lang="bash">
            mkinitcpio -P
        </code-block>
        <warning>
            <p><b>Important:</b> Failing to complete this step will result in a boot failure, as the system will be unable to locate the encrypted root partition on the USB drive.</p>
</warning>
    </step>
    <step>
        <p><b>Fix 2: Handle USB Timing.</b></p>
        <p>Get the UUID of the encrypted partition (<code>sdb2</code>).</p>
        <code-block lang="bash">
            lsblk -f
        </code-block>
    </step>
    <step>
        <p>Edit the bootloader config at <code>/boot/loader/entries/arch.conf</code>.</p>
        <list>
            <li>Add the <code>rd.luks.name</code> parameter with the UUID.</li>
            <li>Add <code>rootwait</code> to the end of the options line to prevent timeouts while the USB drive spins up.</li>
        </list>
        <code-block lang="bash">
            # /boot/loader/entries/arch.conf
            title   Arch Linux
            linux   /vmlinuz-linux
            initrd  /amd-ucode.img
            initrd  /initramfs-linux.img
            options rd.luks.name=YOUR-UUID-HERE=cryptroot root=/dev/mapper/cryptroot rw rootwait
        </code-block>
    To get YOUR-UUID run the following command (replace /dev/sdb2 with your root partition):
    <code-block lang="bash">
        blkid /dev/sdb2
    </code-block>
    </step>
    <step>
        <p>Exit chroot and reboot to test the password prompt.</p>
    </step>
</procedure>

## TPM2 Auto-Unlock & Secure Boot

<procedure title="Phase 3: TPM2 Auto-Unlock &amp; Secure Boot" id="phase-3-tpm">
    <step>
        <p>Install TPM2 tools.</p>
        <code-block lang="bash">
            sudo pacman -S tpm2-tools
        </code-block>
    </step>
    <step>
        <p>Enroll the TPM chip into the LUKS header. Bind it to PCR 7 (Secure Boot state).</p>
        <code-block lang="bash">
            sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=7 /dev/sdb2
        </code-block>
    </step>
    <step>
        <p>Verify the token was added.</p>
        <code-block lang="bash">
            sudo cryptsetup luksDump /dev/sdb2
        </code-block>
        <p><i>Look for "systemd-tpm2" under the Tokens section.</i></p>
    </step>
    <step>
        <p>Update the bootloader to actively check for the TPM device.</p>
        <p>Edit <code>/boot/loader/entries/arch.conf</code> and add <code>rd.luks.options=tpm2-device=auto</code>.</p>
        <code-block lang="bash">
            # Final Options Line
            options rd.luks.name=YOUR-UUID-HERE=cryptroot rd.luks.options=tpm2-device=auto root=/dev/mapper/cryptroot rw rootwait
        </code-block>
    </step>
    <step>
        <p>Reboot. The system should now boot without asking for a password, provided Secure Boot is active and the drive is connected to the enrolled motherboard.</p>
    </step>
</procedure>