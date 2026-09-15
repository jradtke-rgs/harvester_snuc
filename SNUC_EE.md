# SNUC EE Series

Notes for installing Harvester on SNUC EE-series hardware. The install
procedure, console workaround, and known issues are common across the
SNUC EE line; only Harvester version compatibility currently differs by
model. Model-specific sections are called out below and will grow as
more models are tested.

## High-level steps

1. Update BIOS
2. Set date/time at BIOS
3. Boot to USB and modify the GRUB entry (`console=ttyS0,115200n8 nomodeset`)

## Console Settings

- **BIOS** — you need to update the Console settings in the BIOS.
- **grub.cfg** — you need to pass additional parameters to the boot string for the lack of default graphics head.

## Installation Choices

- Virtual Media vs USB Drive
- PXE (future)

## Supported Harvester Versions by Model

### SNUC EE-2300

| Harvester Release | Works | Notes |
|:------------------|:-----:|:------|
| 1.8.2 | Y | Kernel 6.x |
| 1.7.1 | N (important) | Kernel 6.x |
| 1.6.x | N (haven't attempted) | Kernel v5. |

### SNUC EE-8700

| Harvester Release | Works | Notes |
|:------------------|:-----:|:------|
| 1.8.2 | TBD | Kernel 6.x |
| 1.7.1 | Y | Kernel 6.x |
| 1.6.x | N | Kernel v5. |

> [!NOTE]
> We have tested these versions - your experience may differ. Contact your account team if you run in to issues.

## Known Issues

- **Installer can't proceed past network.**
  - For some reason the TUI does not display the network page correctly. This may preclude you from seeing the "MTU value" altogether. You can attempt to fill in the values you can see in that page and simply hit 'enter' to proceed to the next page. If that does not work, the version of harvester you are attemping may not work with the hardware version you have.
- **No display on console** - if you do not update GRUB at boot time, you likely will not see any console output.

### Adjust fan speed
```
To temporarily switch off or adjust the fans for the rack mount kits, you can manually set the PWM speed via the BMC using UART or SSH.
Everything is below is supposed to be apply in the BMC via UART or SSH

Here is how you can do it:
1. Stop the Automatic Control Service
First, disable the automated fan control service so your manual settings aren't overridden:
systemctl stop phosphor-pid-control.service

2. Set the Manual Fan Speed
Next, apply your desired manual PWM speed. The value can be anywhere between 0 (off) and 255 (maximum speed). Assuming you want to switch them off completely, you can use 0.

Run the following command, replacing {PWM} with your desired value:
for f in /sys/class/hwmon/hwmon*/pwm[0-9]*; do echo {PWM} > "$f"; done

Example:
If you wanted to set the speed to 150, you would run:
for f in /sys/class/hwmon/hwmon*/pwm[0-9]*; do echo 150 > "$f"; done

3. Restore Automatic Control
This change is not persistent so if you reboot, the settings will revert to normal, or you can run:
systemctl start phosphor-pid-control.service
```
