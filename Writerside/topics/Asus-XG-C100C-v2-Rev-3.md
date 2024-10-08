# Asus XG-C100C v2 Rev 3

## Driver Notes

It seems that the Asus product page for the XG-C100C v2 Rev 3 has a [old driver](https://www.asus.com/networking-iot-servers/wired-networking/all-series/xg-c100c/helpdesk_download?model2Name=XG-C100C) from back in 2022.

Looking at the device and vendor id in Windows:

![XG-C100C-vendor_id.png](XG-C100C-vendor_id.png)

The device id is `94C0` and the vendor id is `1D6A`, which according to [this blog](https://rog-forum.asus.com/t5/gaming-network-adaptors/xg-c100c-v2-rev-3-flawed-detection-on-reboots-amp-cold-boots/td-p/912130)
translates to "1D6A" which is "Antigua" and a device id of "94C0 ". the device id 94 is a "AQC113CS".

## Latest Driver for AQC113CS chipset

Looking at the [Marvell](https://www.marvell.com/support/downloads.html) website, the latest driver for the AQC113CS is 3.1.1.0, which was released on 23/04/2024.


<note>
Post downloading, extracting and installing. This driver seems to be compatible with Windows 11 24H2.
</note>