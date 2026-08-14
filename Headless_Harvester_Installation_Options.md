**RGS Solutions Architecture --- Technical Note**

Installing RGS Harvester for Government on Headless Systems

*Options for console access and an actionable workaround for
text-console-only hardware*

Purpose

This note documents the options for installing RGS Harvester for
Government on servers with no VGA output --- edge appliances, rack
servers accessed only through a BMC, or any target where a monitor
cannot be physically or remotely attached. It also captures a concrete,
tested workaround for driving the interactive installer over a
serial/text console, since this is not currently documented upstream.

This guidance is hardware-agnostic. It applies to any headless target
--- edge compute appliances, rack-mount servers, or cloud/bare-metal
instances --- not to a specific vendor or model.

The problem

The Harvester installer (harvester-installer) always attaches its
interactive TUI, and the post-install Dashboard, to whichever console it
detects as tty1 --- effectively assuming a VGA/graphical console is
present. On a system with no VGA output, and no BMC feature that
emulates one, the installer output is invisible: it is running, but
nothing is displayed anywhere the operator can reach.

This is a known, long-standing limitation (see GitHub Issues, below),
not a misconfiguration on the operator\'s part. It affects any target
whose only console is serial (ttyS0/SOL) rather than VGA.

Options, in order of preference

Evaluate these in the order below for any given target. Preference is
driven by how much manual, interactive work is required and how
repeatable the result is.

  -----------------------------------------------------------------------------
  **Option**    **What it requires**  **Pros**           **Cons**
  ------------- --------------------- ------------------ ----------------------
  BMC /         BMC with full video   Installer TUI      Requires confirming
  KVM-over-IP   (KVM) redirection,    works exactly as   the BMC tier actually
  virtual       e.g. IPMI, Redfish,   on a monitor; no   includes video/KVM,
  console       iDRAC, iLO, or a      kernel arg or      not just power control
                vendor-specific       config changes     and
                out-of-band           needed; supports   text/serial-over-LAN
                controller with       virtual media to   
                graphical console     mount the ISO      
                support               remotely           

  PXE boot with Network boot          Fully              Requires PXE
  config.yaml   infrastructure        non-interactive;   infrastructure; less
                (DHCP/TFTP or iPXE)   no console         useful for a single
                and a prepared        redirection needed one-off box with no
                config.yaml           at all; repeatable network boot
                                      and scriptable for environment
                                      fleet installs     

  Automatic     Ability to edit       No PXE server      Requires hosting the
  install via   boot/kernel           required; works    config file somewhere
  kernel        parameters (GRUB,     from an ISO or raw reachable at boot
  arguments     iPXE script, or       disk image; fully  time; more setup than
                IPMI-set boot         unattended         a simple ISO boot
                options) plus a                          
                hosted config_url                        

  Serial        Physical or           Works on hardware  Manual, interactive,
  console       BMC-provided          with no VGA output undocumented upstream;
  workaround    serial/text console   and no KVM-capable sensitive to terminal
  (GRUB         access (SOL, minicom, BMC; uses only a   sizing; not scriptable
  console=      or a null-modem       text console       for repeat installs
  edit)         connection)                              
  -----------------------------------------------------------------------------

1\. BMC / KVM-over-IP virtual console (preferred when available)

If the target\'s BMC provides true video/KVM redirection (not just power
control or text-based serial-over-LAN), this sidesteps the whole
problem: the installer renders exactly as it would on an attached
monitor, and virtual media can be used to mount the Harvester ISO
without physically touching the box.

Before assuming this is available, confirm with the hardware vendor
whether the BMC tier on the specific model includes full KVM video, or
only power/health monitoring and a text console. Some vendor BMC feature
sets vary by model within the same product line --- do not assume KVM
support carries across the whole lineup.

2\. PXE boot with config.yaml (preferred for repeatable / fleet
installs)

When booting via PXE, the interactive installer is bypassed entirely ---
Harvester is configured from a supplied config.yaml instead. This
removes the console problem altogether, since there is no TUI to
display. This is the most robust option for customer deployments where
the install needs to be repeatable or scripted, and it works regardless
of whether the target has any usable console at all.

Reference: RGS/Harvester PXE boot install documentation for the
config.yaml schema.

3\. Automatic install via kernel arguments

Similar in spirit to PXE, but usable from a plain ISO boot or a raw disk
image without standing up PXE infrastructure. Setting
harvester.install.automatic=true along with
harvester.install.config_url=\<url-to-config.yaml\> at the kernel
command line drives a fully unattended install. The config file (or the
kernel args directly) can also set install.tty to point logging at a
specific serial device, e.g.:

install: tty: ttyS0,115200n8

This is a good middle ground: no PXE server needed, but still fully
unattended and scriptable, which matters for FIPS/STIG-documented,
repeatable install procedures.

4\. Serial console workaround (manual, interactive --- use only when the
above are not available)

If the target truly has no VGA, no KVM-capable BMC, and no network boot
path (e.g. a single appliance being staged by hand), the installer can
still be driven manually over a serial console by redirecting it there
at boot. This is the workaround captured in GitHub Issue #5637 and is
not part of the official Harvester documentation.

Actionable workaround: driving the installer over a serial console

Use this procedure when Options 1--3 above are not available for the
target hardware.

1.  Connect to the target\'s serial console (physical null-modem cable,
    USB-serial adapter, or the BMC\'s serial-over-LAN / SOL feature)
    using a terminal program such as minicom or screen.

2.  Boot the Harvester ISO and interrupt at the GRUB menu (press Esc to
    stay on the menu when it appears).

3.  Press e on the first menu entry to edit it.

4.  Locate the kernel command line and append the console parameter for
    your serial device and baud rate, for example:

console=ttyS0,115200n8

Adjust the device name (ttyS0, ttyS4, etc.) to match the port your BMC
or hardware actually exposes --- this varies by vendor and, on
multi-UART boards, by which header/port is wired to the accessible
connector. If an existing console=tty1 (VGA) entry is already present,
you can leave it in place and simply add the serial entry after it; the
last console= listed becomes the primary /dev/console.

1.  Press Ctrl+X (or F10, depending on the GRUB build) to boot with the
    edited line.

2.  In the serial terminal, maximize the terminal window so it has
    reasonable dimensions --- the installer TUI can panic with an
    "invalid dimensions" error if the terminal is too small or too
    short.

3.  Log in with the default credentials (rancher / rancher), then
    elevate:

sudo su -

1.  Resize the terminal so the TUI renders correctly:

setterm \--resize

1.  Manually launch the installer:

start-installer.sh

1.  Proceed through the interactive installer as normal (disk selection,
    network configuration, cluster token, etc.).

2.  Once installation completes and the system reboots, remove the
    installation media (USB/virtual ISO) --- otherwise some systems will
    boot back into the installer rather than the installed disk.

Notes and gotchas

- The console= kernel parameter can be repeated, but only once per
  console technology (e.g. console=tty0 console=ttyS0 is valid;
  console=ttyS0 console=ttyS1 is not). Whichever console is listed last
  becomes the primary /dev/console and receives keyboard input.

- This workaround is manual and interactive by nature --- it is a
  reasonable fallback for one-off staging of a single appliance, but it
  is not a substitute for Option 2 or 3 when the install needs to be
  repeatable, scripted, or documented as part of a FIPS/STIG-compliant
  build procedure.

- After installation, the post-install Dashboard is also tied to tty1 by
  default per Issue #485 --- if ongoing console access to the running
  node (as opposed to just the installer) is needed, the same console=
  redirection approach should be re-applied to the installed system\'s
  boot configuration, not just the installer boot.

Related GitHub issues

These issues (in harvester/harvester) document the underlying limitation
and the source of the workaround above.

  ----------------------------------------------------------------------------------
  **Issue**             **Summary**                     **Status / relevance**
  --------------------- ------------------------------- ----------------------------
  harvester/harvester   Original bug report: the        Root-cause report
  #485                  installer and post-install      establishing the underlying
                        dashboard are always tied to    limitation. No native fix;
                        tty1 (VGA), even on headless    behavior persists.
                        servers or cloud instances      
                        (Equinix Metal, KVM/virsh) that 
                        only expose a serial console.   

  harvester/harvester   User question: reached the      Closed as a question, not a
  #3393                 ttyS0 login prompt (SUSE Linux  bug fix. Confirms the gap is
                        Enterprise Micro / rancher      a documentation gap, not
                        login) but had no documented    just a code limitation.
                        way to drive the interactive    
                        installer TUI from that         
                        console.                        

  harvester/harvester   Documentation request that also Tagged require/doc. This is
  #5637                 contains the community-sourced  the source of the actionable
                        workaround: edit the GRUB entry workaround below; it has not
                        to add                          been merged into the
                        console=ttyS\<N\>,115200,       official docs as of this
                        resize the terminal, then       writing.
                        manually run the installer.     
  ----------------------------------------------------------------------------------

Links

• [Issue #485 --- The installer and dashboard are always displayed on
tty1](https://github.com/harvester/harvester/issues/485)

• [Issue #3393 --- Can you provide instructions to install harvester
without VGA
(headless)](https://github.com/harvester/harvester/issues/3393)

• [Issue #5637 --- \[DOC\] Tips to install Harvester with a serial
console](https://github.com/harvester/harvester/issues/5637)

Recommendation

For customer engagements, lead with Option 1 (BMC/KVM virtual console)
if the hardware vendor confirms it\'s available, since it requires no
deviation from the standard ISO-boot procedure. For any deployment
intended to be repeatable, documented, or delivered as part of a runbook
--- which is the common case for RGS federal/DoD customer engagements
--- prefer Option 2 or 3 (PXE or automatic install with config.yaml)
regardless of whether a console is available at all, since they remove
the interactive installer from the picture entirely. Reserve the serial
console workaround (Option 4) for one-off staging where no BMC video and
no network boot path exist.
