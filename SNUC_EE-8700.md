# SNUC_8700

## High-level steps
Update BIOS
Set date/time at BIOS
Boot to USB and modify grub entry ("console=ttyS0,115200n8 nomodest)

## Console Settings
- BIOS - you need to update the Console settings in the BIOS
- grub.cfg - you need to pass additional parameters to the boot string for the lack of default graphics head.

## Installation Choices
- Virtual Media vs USB Drive
- PXE (future)

## Supported Harvester Versions
| Harvester Release | Works | Notes |
|:------------------|:-----:|:------|
| 1.8.2 | TBD | Kernel 6.x |
| 1.7.1 | Y | Kernel 6.x |
| 1.6.x | N | Kernel v5. |

> [!NOTE]
> We have tested these versions - your experience may differ.  Contact your account team if you run in to issues.

# known issues:
- Installer can't proceed past network.
 - For some reason the TUI does not display the network page correctly.  This may preclude you from seeing the "MTU value" altogether.  You can attempt to fill in the values you can see in that page and simply hit 'enter' to proceed to the next page.  If that does not work, the version of harvester you are attemping may not work with the hardware version you have.
- No display on console - if you do not update GRUB at boot time, you likely will not see any console output


# References
https://docs.harvesterhci.io/v1.8/troubleshooting/os/#how-to-change-the-default-grub-boot-menu-entry  
https://docs.harvesterhci.io/v1.8/install/usb-install  
https://docs.harvesterhci.io/v1.8/install/usb-install#graphics-issue explains how to update grub at install time for headless systems  

## nomodeset - grub option
https://en.opensuse.org/SDB:Nomodeset:_Work_Around_Graphic_Upgrade_%26_Installation_Obstacles - nomodeset option  
https://documentation.suse.com/smart/systems-management/html/kernel-boot-parameters-modify/index.html
