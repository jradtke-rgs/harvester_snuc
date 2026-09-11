# SNUC_2300

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
| 1.8.2 | Y | Kernel 6.x |
| 1.7.1 | N (important) | Kernel 6.x |
| 1.6.x | N (haven't attempted) | Kernel v5. |

> [!NOTE]
> We have tested these versions - your experience may differ.  Contact your account team if you run in to issues.

# known issues:
- Installer can't proceed past network.
  - For some reason the TUI does not display the network page correctly.  This may preclude you from seeing the "MTU value" altogether.  You can attempt to fill in the values you can see in that page and simply hit 'enter' to proceed to the next page.  If that does not work, the version of harvester you are attemping may not work with the hardware version you have.
- No display on console - if you do not update GRUB at boot time, you likely will not see any console output

