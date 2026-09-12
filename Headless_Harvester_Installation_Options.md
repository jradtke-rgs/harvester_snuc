# Headless Harvester Installation Options

**RGS Solutions Architecture — Technical Note**

## Installing RGS Harvester for Government on Headless Systems

*Options for console access and an actionable workaround for text-console-only hardware*

## Purpose

This note documents the options for installing RGS Harvester for Government on servers with no VGA output — edge appliances, rack servers accessed only through a BMC, or any target where a monitor cannot be physically or remotely attached. It also captures a concrete, tested workaround for driving the interactive installer over a serial/text console, since this is not currently documented upstream.

This guidance is hardware-agnostic. It applies to any headless target — edge compute appliances, rack-mount servers, or cloud/bare-metal instances — not to a specific vendor or model.

## The problem

The Harvester installer (`harvester-installer`) always attaches its interactive TUI, and the post-install Dashboard, to whichever console it detects as `tty1` — effectively assuming a VGA/graphical console is present. On a system with no VGA output, and no BMC feature that emulates one, the installer output is invisible: it is running, but nothing is displayed anywhere the operator can reach.

This is a known, long-standing limitation (see GitHub Issues, below), not a misconfiguration on the operator's part. It affects any target whose only console is serial (`ttyS0`/SOL) rather than VGA.

## Options, in order of preference

Evaluate these in the order below for any given target. Preference is driven by how much manual, interactive work is required and how repeatable the result is.

| Option | What it requires | Pros | Cons |
| --- | --- | --- | --- |
| **1. BMC / KVM-over-IP virtual console** | BMC with full video (KVM) redirection, e.g. IPMI, Redfish, iDRAC, iLO, or a vendor-specific out-of-band controller with graphical console support | Installer TUI works exactly as on a monitor; no kernel arg or config changes needed; supports virtual media to mount the ISO remotely | Requires confirming the BMC tier actually includes video/KVM, not just power control and text/serial-over-LAN |
| **2. PXE boot with config.yaml** | Network boot infrastructure (DHCP/TFTP or iPXE) and a prepared `config.yaml` | Fully non-interactive; no console redirection needed at all; repeatable and scriptable for fleet installs | Requires PXE infrastructure; less useful for a single one-off box with no network boot environment |
| **3. Automatic install via kernel arguments** | Ability to edit boot/kernel parameters (GRUB, iPXE script, or IPMI-set boot options) plus a hosted `config_url` | No PXE server required; works from an ISO or raw disk image; fully unattended | Requires hosting the config file somewhere reachable at boot time; more setup than a simple ISO boot |
| **4. Serial console workaround** (GRUB `console=` edit) | Physical or BMC-provided serial/text console access (SOL, minicom, or a null-modem connection) | Works on hardware with no VGA output and no KVM-capable BMC; uses only a text console | Manual, interactive, undocumented upstream; sensitive to terminal sizing; not scriptable for repeat installs |

### 1. BMC / KVM-over-IP virtual console (preferred when available)

If the target's BMC provides true video/KVM redirection (not just power control or text-based serial-over-LAN), this sidesteps the whole problem: the installer renders exactly as it would on an attached monitor, and virtual media can be used to mount the Harvester ISO without physically touching the box.

Before assuming this is available, confirm with the hardware vendor whether the BMC tier on the specific model includes full KVM video, or only power/health monitoring and a text console. Some vendor BMC feature sets vary by model within the same product line — do not assume KVM support carries across the whole lineup.

### 2. PXE boot with config.yaml (preferred for repeatable / fleet installs)

When booting via PXE, the interactive installer is bypassed entirely — Harvester is configured from a supplied `config.yaml` instead. This removes the console problem altogether, since there is no TUI to display. This is the most robust option for customer deployments where the install needs to be repeatable or scripted, and it works regardless of whether the target has any usable console at all.

Reference: RGS/Harvester PXE boot install documentation for the `config.yaml` schema.

### 3. Automatic install via kernel arguments

Similar in spirit to PXE, but usable from a plain ISO boot or a raw disk image without standing up PXE infrastructure. Setting `harvester.install.automatic=true` along with `harvester.install.config_url=<url-to-config.yaml>` at the kernel command line drives a fully unattended install. The config file (or the kernel args directly) can also set `install.tty` to point logging at a specific serial device, e.g.:

```
install:
  tty: ttyS0,115200n8
```

This is a good middle ground: no PXE server needed, but still fully unattended and scriptable, which matters for FIPS/STIG-documented, repeatable install procedures.

### 4. Serial console workaround (manual, interactive — use only when the above are not available)

If the target truly has no VGA, no KVM-capable BMC, and no network boot path (e.g. a single appliance being staged by hand), the installer can still be driven manually over a serial console by redirecting it there at boot. This is the workaround captured in GitHub Issue #5637 and is not part of the official Harvester documentation.

## Actionable workaround: driving the installer over a serial console

Use this procedure when Options 1–3 above are not available for the target hardware.

1. Connect to the target's serial console (physical null-modem cable, USB-serial adapter, or the BMC's serial-over-LAN / SOL feature) using a terminal program such as minicom or screen.
2. Boot the Harvester ISO and interrupt at the GRUB menu (press Esc to stay on the menu when it appears).
3. Press `e` on the first menu entry to edit it.
4. Locate the kernel command line and append the console parameter for your serial device and baud rate, for example:

   ```
   console=ttyS0,115200n8
   ```

   Adjust the device name (`ttyS0`, `ttyS4`, etc.) to match the port your BMC or hardware actually exposes — this varies by vendor and, on multi-UART boards, by which header/port is wired to the accessible connector. If an existing `console=tty1` (VGA) entry is already present, you can leave it in place and simply add the serial entry after it; the last `console=` listed becomes the primary `/dev/console`.

5. Press Ctrl+X (or F10, depending on the GRUB build) to boot with the edited line.
6. In the serial terminal, maximize the terminal window so it has reasonable dimensions — the installer TUI can panic with an "invalid dimensions" error if the terminal is too small or too short.
7. Log in with the default credentials (`rancher` / `rancher`), then elevate:

   ```
   sudo su -
   ```

8. Resize the terminal so the TUI renders correctly:

   ```
   setterm --resize
   ```

9. Manually launch the installer:

   ```
   start-installer.sh
   ```

10. Proceed through the interactive installer as normal (disk selection, network configuration, cluster token, etc.).
11. Once installation completes and the system reboots, remove the installation media (USB/virtual ISO) — otherwise some systems will boot back into the installer rather than the installed disk.

## Notes and gotchas

- The `console=` kernel parameter can be repeated, but only once per console technology (e.g. `console=tty0 console=ttyS0` is valid; `console=ttyS0 console=ttyS1` is not). Whichever console is listed last becomes the primary `/dev/console` and receives keyboard input.
- This workaround is manual and interactive by nature — it is a reasonable fallback for one-off staging of a single appliance, but it is not a substitute for Option 2 or 3 when the install needs to be repeatable, scripted, or documented as part of a FIPS/STIG-compliant build procedure.
- After installation, the post-install Dashboard is also tied to `tty1` by default per Issue #485 — if ongoing console access to the running node (as opposed to just the installer) is needed, the same `console=` redirection approach should be re-applied to the installed system's boot configuration, not just the installer boot.

## Related GitHub issues

These issues (in harvester/harvester) document the underlying limitation and the source of the workaround above.

| Issue | Summary | Status / relevance |
| --- | --- | --- |
| [harvester/harvester #485](https://github.com/harvester/harvester/issues/485) | Original bug report: the installer and post-install dashboard are always tied to `tty1` (VGA), even on headless servers or cloud instances (Equinix Metal, KVM/virsh) that only expose a serial console. | Root-cause report establishing the underlying limitation. No native fix; behavior persists. |
| [harvester/harvester #3393](https://github.com/harvester/harvester/issues/3393) | User question: reached the `ttyS0` login prompt (SUSE Linux Enterprise Micro / rancher login) but had no documented way to drive the interactive installer TUI from that console. | Closed as a question, not a bug fix. Confirms the gap is a documentation gap, not just a code limitation. |
| [harvester/harvester #5637](https://github.com/harvester/harvester/issues/5637) | Documentation request that also contains the community-sourced workaround: edit the GRUB entry to add `console=ttyS<N>,115200`, resize the terminal, then manually run the installer. | Tagged require/doc. This is the source of the actionable workaround below; it has not been merged into the official docs as of this writing. |

## Links

- [Issue #485 — The installer and dashboard are always displayed on tty1](https://github.com/harvester/harvester/issues/485)
- [Issue #3393 — Can you provide instructions to install harvester without VGA (headless)](https://github.com/harvester/harvester/issues/3393)
- [Issue #5637 — \[DOC\] Tips to install Harvester with a serial console](https://github.com/harvester/harvester/issues/5637)

## Recommendation

For customer engagements, lead with Option 1 (BMC/KVM virtual console) if the hardware vendor confirms it's available, since it requires no deviation from the standard ISO-boot procedure. For any deployment intended to be repeatable, documented, or delivered as part of a runbook — which is the common case for RGS federal/DoD customer engagements — prefer Option 2 or 3 (PXE or automatic install with config.yaml) regardless of whether a console is available at all, since they remove the interactive installer from the picture entirely. Reserve the serial console workaround (Option 4) for one-off staging where no BMC video and no network boot path exist.
