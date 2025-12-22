# Secure Drive Setup


## Securing Arch Linux on External USB with LUKS & TPM2

This guide documents the conversion of an existing Arch Linux installation on an external Samsung T7 SSD to a fully encrypted setup with Secure Boot and TPM2 auto-unlocking.

## Prerequisites
- Existing Arch Linux installation on an external USB drive (e.g., Samsung T7).
- Arch Linux Live ISO for recovery and setup.
- Basic familiarity with LUKS encryption, systemd-boot, and TPM2.


## Backup & Encryption

<procedure title="Phase 1: Backup and Encryption" id="phase-1-encryption">
    <step>
        <p>Boot into the Arch Linux Live ISO.</p>
    </step>
    <step>
        <p>Create a backup of the existing system (if not already done) using <code>rsync</code> to a separate drive.</p>
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
            rsync -aAXv /path/to/backup/ /mnt/ --exclude=/path/to/backup
        </code-block>
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