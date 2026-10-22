### Sandboxing for testing

Tart and UTM are virtual machine (VM) managers for Apple Silicon Macs. A VM is a whole second computer, with its own OS, disk and network, running inside the host Mac. That lets Guardian change DNS settings in the VM without touching the host Mac.

Both run on the hypervisor built into macOS. How a program in the guest VM runs on the host's chip:

```txt
Guest OS (macOS, Linux, Windows)
  → virtual hardware: Apple's Virtualization.framework, or QEMU (UTM only)
  → Hypervisor.framework, macOS's built-in hypervisor
  → Apple Silicon CPU, which runs the guest's code directly
```

#### Tart

- What it is: a command-line VM tool from Cirrus Labs. Every step is a command, so it can be scripted.
- Hypervisor: Apple's Virtualization.framework only. That supports macOS and Linux guests, so no Windows.
- Starting file: a ready-made, already-installed image pulled from a registry (`ghcr.io/cirruslabs/macos-sequoia-vanilla`). It's a 23.7 GB download in 96 compressed chunks.
- Files in a VM:
  - `config.json` (509 B): CPUs, memory, display, the virtual Mac's hardware ID
  - `nvram.bin` (32 MB): startup settings, like a Mac's NVRAM
  - `disk.img` (50 GB max, ~28 GB used): the installed macOS, with a user and SSH already set up
- Where: the downloaded image is in `~/.tart/cache`, VMs are in `~/.tart/vms`. A VM is an APFS copy of the image, so the two share most of their space (~34 GB together, not 60).
- New VM: `tart clone` the image. Nothing to install.
- Use for: the Guardian macOS sandbox (`scripts/guardian-sandbox.sh`).

#### UTM

- What it is: an open-source Mac app for VMs, used through its GUI rather than from a script.
- Hypervisor: two back ends. Apple Virtualization (the same as Tart) for macOS and Linux, and QEMU for everything else. QEMU runs ARM guests like Windows 11 ARM at near-native speed through Hypervisor.framework, and emulates x86 guests in software, which is slow.
- Starting file: an installer, not an installed system: a Windows `.iso` (7.9 GB for Windows 11 ARM) or a macOS `.ipsw` restore image (~15–20 GB).
- Files in a VM: a `.utm` bundle (a folder that Finder shows as one file):
  - `config.plist`: settings
  - startup settings for the virtual firmware
  - the virtual disk (`.qcow2` for QEMU, `.img` for Apple): grows to 20–30 GB once the OS is installed
- Where: `~/Library/Containers/com.utmapp.UTM/Data/Documents`.
- New VM: install the OS from the starting file by hand, or clone a VM you set up earlier.
- Use for: Windows testing.

|             | **Tart**                | **UTM**               |
| ----------- | ----------------------- | --------------------- |
| **OS**      | macOS, Linux            | macOS, Linux, Windows |
| **Setup**   | Scripted                | By hand, in the app   |
| **New VM**  | Pull a ready-made image | Clone one you set up  |
| **Use for** | Guardian macOS sandbox  | Windows testing       |
