# Booting Issues with Samsung T7

<tip>
<b>Status: Under Investigation</b>
</tip>

## Overview

Recently since updating the BIOS to version: `PRIME-X670E-PRO-WIFI-ASUS-3287` there seems to be a timing issue.
When the machine is powered on and the BIOS is initialising, then most times it fails to detect the T7 and thus
the boot order changes with the only detected drive being the internal NVMe drive which has a EFI partition for Windows only.

This results in the machine failing to show the systemd-boot menu that would allow me to choose between Windows
and Arch Linux.

## Symptoms

- BIOS fails to detect Samsung T7 USB drive during POST approximately 70% of boot attempts
- systemd-boot menu does not appear when T7 is not detected
- Only Windows EFI partition (on internal NVMe) is accessible
- Issue began after BIOS update to version `PRIME-X670E-PRO-WIFI-ASUS-3287`

## Environment

- **Motherboard**: ASUS PRIME X670E-PRO WIFI
- **BIOS Version**: 3287
- **USB Drive**: Samsung T7
- **Boot Manager**: systemd-boot
- **Operating Systems**: Arch Linux (T7), Windows (internal NVMe)

## Root Cause Analysis

### Hypothesis 1: USB Initialization Timing
The BIOS POST process completes USB enumeration before the T7 is ready to respond.

**Evidence**:
- Issue is intermittent (~70% failure rate)
- T7 was consistently detected after Secure Boot key resolution
- Fast Boot delay settings have no effect

**Tested Solutions**:
<procedure>
<step number="1" title="Increase BIOS Boot Delay">
Increased delay at:
<code-block>
Advanced > Boot > Fast Boot > Fast Boot Delay Time
</code-block>
<b>Result</b>: No improvement
</step>

<step number="2" title="Disable Fast Boot">
Disabled Fast Boot at:
<code-block>
Advanced > Boot > Fast Boot > Fast Boot
</code-block>
<b>Result</b>: No improvement
</step>
</procedure>

### Hypothesis 2: BIOS Regression
The 3287 BIOS update introduced a timing regression in USB device enumeration.

**Evidence**:
- Issue began immediately after BIOS update
- Previous BIOS version did not exhibit this behavior

## Workarounds

<note>
None currently identified. Manual BIOS entry and boot order selection required on failed boots. When the BIOS
detects the T7 (seems to work 1 out of 3 attempts).
</note>

## Next Steps

1. Test previous BIOS version to confirm regression
2. Check for BIOS microcode/AGESA updates
3. Test alternative USB ports (direct motherboard vs. front panel)
4. Monitor Samsung T7 firmware version
5. Contact ASUS support with findings

## Related Issues

- [Secure Boot Configuration](Secureboot.md) — Previously resolved Secure Boot key issue

## Timeline

- **2025-12-17**: BIOS updated to 3287
- **2025-12-19**: Secure Boot keys resolved, T7 detection stable
- **2025-12-20**: Intermittent T7 detection failures began

## References

- [ASUS PRIME X670E-PRO WIFI Support](https://www.asus.com/motherboards-components/motherboards/prime/prime-x670e-pro-wifi/helpdesk_bios/)
- [Samsung T7 Specifications](https://www.samsung.com/semiconductor/minisite/ssd/product/portable/t7/)
