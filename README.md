# Windows 7 on Android — Termux + QEMU + RealVNC (No Root Required)

> **Master README:** This document covers the complete phone-only workflow from a clean Android/Termux setup to a running Windows 7 VM, including VNC access, networking, normal shutdown, startup scripts, backup, and troubleshooting for the problems encountered during earlier setup attempts.

## Table of contents

- [Quick map](#quick-map)
- [2. Requirements](#2-requirements)
- [3. Install the correct Termux](#3-install-the-correct-termux)
- [4. Update Termux](#4-update-termux)
- [5. Give Termux access to phone storage](#5-give-termux-access-to-phone-storage)
- [6. Install QEMU](#6-install-qemu)
- [7. Install RealVNC Viewer](#7-install-realvnc-viewer)
- [8. Get the Windows 7 ISO](#8-get-the-windows-7-iso)
- [9. Create a folder for the Windows 7 VM](#9-create-a-folder-for-the-windows-7-vm)
- [10. Copy the Windows 7 ISO into the VM folder](#10-copy-the-windows-7-iso-into-the-vm-folder)
- [11. Create the virtual hard disk](#11-create-the-virtual-hard-disk)
- [12. Test the QEMU VNC capability](#12-test-the-qemu-vnc-capability)
- [13. First boot: Windows 7 installation](#13-first-boot-windows-7-installation)
- [14. Connect RealVNC Viewer to Windows 7](#14-connect-realvnc-viewer-to-windows-7)
- [15. Install Windows 7](#15-install-windows-7)
- [16. First boot after Windows installation](#16-first-boot-after-windows-installation)
- [17. Create a start script](#17-create-a-start-script)
- [18. Create a stop script](#18-create-a-stop-script)
- [19. Run the VM with more or less RAM](#19-run-the-vm-with-more-or-less-ram)
- [20. Change the virtual CPU count](#20-change-the-virtual-cpu-count)
- [21. Improve Windows 7 usability](#21-improve-windows-7-usability)
- [22. Check the VM files](#22-check-the-vm-files)
- [23. Check whether QEMU is running](#23-check-whether-qemu-is-running)
- [24. Check whether VNC port 5901 is listening](#24-check-whether-vnc-port-5901-is-listening)
- [25. RealVNC connection fails](#25-realvnc-connection-fails)
- [26. QEMU command not found](#26-qemu-command-not-found)
- [27. `qemu-system-x86_64` crashes or library error appears](#27-qemu-system-x86_64-crashes-or-library-error-appears)
- [28. QEMU installs but the x86_64 executable is missing](#28-qemu-installs-but-the-x86_64-executable-is-missing)
- [29. Windows 7 freezes during boot](#29-windows-7-freezes-during-boot)
- [30. Windows installer cannot see the disk](#30-windows-installer-cannot-see-the-disk)
- [31. Mouse is difficult to control](#31-mouse-is-difficult-to-control)
- [32. Windows 7 screen is black](#32-windows-7-screen-is-black)
- [33. Android kills Termux while Windows is running](#33-android-kills-termux-while-windows-is-running)
- [34. Phone gets very hot](#34-phone-gets-very-hot)
- [35. Windows 7 is extremely slow](#35-windows-7-is-extremely-slow)
- [36. Network inside Windows 7](#36-network-inside-windows-7)
- [37. Do not expose the VNC server to the network unless you know what you are doing](#37-do-not-expose-the-vnc-server-to-the-network-unless-you-know-what-you-are-doing)
- [38. Optional: VNC password](#38-optional-vnc-password)
- [39. Normal daily workflow](#39-normal-daily-workflow)
- [40. Quick start commands](#40-quick-start-commands)
- [41. Complete folder layout](#41-complete-folder-layout)
- [42. Backup the Windows 7 VM](#42-backup-the-windows-7-vm)
- [43. Restore the VM](#43-restore-the-vm)
- [44. Important limitations](#44-important-limitations)
- [45. One-command diagnostic collection](#45-one-command-diagnostic-collection)
- [46. Sources / further reading](#46-sources-further-reading)
- [47. Final checklist](#47-final-checklist)
- [Note about commands](#note-about-commands)

---

## Quick map

```text
Android Phone
   │
   ├── Termux
   │     └── QEMU x86/x86_64 emulation
   │            └── Windows 7
   │                   └── QEMU VNC :1 → TCP 5901
   │
   └── RealVNC Viewer
          └── 127.0.0.1::5901
```

## 2. Requirements

Recommended starting point:

- Android phone with **64-bit ARM (ARM64)** support
- At least **4 GB RAM**; 6–8 GB or more is preferable
- At least **32 GB free storage**
- Termux from an official source
- RealVNC Viewer from the Google Play Store
- Windows 7 ISO that you are legally licensed to use
- Battery charger recommended
- Wi-Fi is helpful for downloading the ISO/packages

For a 32 GB virtual disk, keep considerably more than 32 GB free because Android, Termux, the ISO, and temporary files also need storage.

---

## 3. Install the correct Termux

Use an official Termux distribution. The Termux project documents **F-Droid and GitHub** as official release sources.

Do not mix Termux and its plugins from different signing sources.

After installation, open Termux.

Check the environment:

```bash
termux-info
```

Check CPU architecture:

```bash
uname -m
```

Also run:

```bash
dpkg --print-architecture
```

On a typical 64-bit ARM phone you should see something such as:

```text
aarch64
```

or:

```text
arm64
```

If your phone reports only a 32-bit ARM environment, QEMU x86_64 support may not work correctly.

---

## 4. Update Termux

Run:

```bash
pkg update
pkg upgrade -y
```

If Termux asks you to choose a repository/mirror, select a working official mirror.

After upgrading, you can check:

```bash
pkg --version
```

---

## 5. Give Termux access to phone storage

Run:

```bash
termux-setup-storage
```

Android will show a permission dialog.

Press **Allow**.

Your Android shared storage becomes available through:

```text
~/storage/
```

Useful folders include:

```text
~/storage/downloads
~/storage/shared
~/storage/pictures
```

Check them:

```bash
ls ~/storage
```

---

## 6. Install QEMU

For an ARM64 Termux environment, install the x86_64 system emulator and disk utility.

Try:

```bash
pkg install qemu-system-x86-64 qemu-utils -y
```

On some Termux package states, the executable is exposed as:

```bash
qemu-system-x86_64
```

Check:

```bash
command -v qemu-system-x86_64
```

Then:

```bash
qemu-system-x86_64 --version
```

If that command is not found, try:

```bash
command -v qemu-system-x86-64
```

and:

```bash
qemu-system-x86-64 --version
```

You can also check what QEMU packages are installed:

```bash
pkg list-installed | grep qemu
```

### Important architecture note

Current Termux QEMU packaging has had problems on some **32-bit ARM** environments. If your Termux architecture is `arm`, do not assume the x86_64 emulator package will work. First confirm `uname -m` and `dpkg --print-architecture`.

---

## 7. Install RealVNC Viewer

Install:

**RealVNC Viewer: Remote Desktop**

You only need the **Viewer** application on Android for this setup because QEMU itself provides the VNC server.

Do not install RealVNC Server inside Windows 7 unless you have a separate reason to do so.

---

## 8. Get the Windows 7 ISO

Use a Windows 7 ISO that you are legally allowed to use.

Do **not** use cracked ISOs, modified ISOs, or activation bypass tools.

Windows 7 is an old, unsupported operating system. Do not use it for modern banking, sensitive accounts, or other security-critical work.

Place the ISO in:

```text
Download
```

For example:

```text
Windows7.iso
```

After downloading, check from Termux:

```bash
ls -lh ~/storage/downloads/
```

You should see your ISO.

---

## 9. Create a folder for the Windows 7 VM

Create a dedicated directory:

```bash
mkdir -p ~/win7
cd ~/win7
```

Check:

```bash
pwd
```

You should get a path similar to:

```text
/data/data/com.termux/files/home/win7
```

---

## 10. Copy the Windows 7 ISO into the VM folder

Example:

```bash
cp ~/storage/downloads/Windows7.iso ~/win7/
```

Check:

```bash
ls -lh ~/win7
```

You should see:

```text
Windows7.iso
```

If your ISO has another name, use the exact filename.

Example:

```bash
cp ~/storage/downloads/en_windows_7.iso ~/win7/
```

---

## 11. Create the virtual hard disk

A QCOW2 disk is a virtual hard drive.

Create a 32 GB disk:

```bash
cd ~/win7
qemu-img create -f qcow2 win7.qcow2 32G
```

Check it:

```bash
qemu-img info win7.qcow2
```

You should see the virtual size around:

```text
32 GiB
```

### Storage sizing

You can create a larger disk when you have enough phone storage.

Examples:

```bash
qemu-img create -f qcow2 win7.qcow2 24G
```

or:

```bash
qemu-img create -f qcow2 win7.qcow2 32G
```

or:

```bash
qemu-img create -f qcow2 win7.qcow2 40G
```

The QCOW2 file is dynamically allocated, so the file does not immediately occupy the entire virtual size.

---

## 12. Test the QEMU VNC capability

Run:

```bash
qemu-system-x86_64 -display help
```

You should see available display modes.

You can also check VNC help:

```bash
qemu-system-x86_64 -help | grep -i vnc
```

If the executable is named differently on your installation, replace the command with:

```bash
qemu-system-x86-64
```

---

## 13. First boot: Windows 7 installation

From the VM directory:

```bash
cd ~/win7
```

Start QEMU with:

```bash
qemu-system-x86_64 \
  -m 2048 \
  -smp 2 \
  -cpu core2duo \
  -machine pc \
  -vga std \
  -device usb-tablet \
  -drive file=win7.qcow2,format=qcow2,if=ide \
  -cdrom Windows7.iso \
  -boot menu=on \
  -nic user,model=rtl8139 \
  -vnc 127.0.0.1:1
```

### What these important options do

```text
-m 2048
```

Gives Windows 7 about 2 GB RAM.

```text
-smp 2
```

Provides two virtual CPU cores.

```text
-cpu core2duo
```

Presents a compatible older x86 CPU model.

```text
-vga std
```

Provides a standard emulated VGA display.

```text
-device usb-tablet
```

Improves mouse positioning when using VNC.

```text
-drive file=win7.qcow2,format=qcow2,if=ide
```

Uses the QCOW2 file as the virtual hard drive.

```text
-cdrom Windows7.iso
```

Mounts the Windows 7 ISO as the virtual CD/DVD.

```text
-nic user,model=rtl8139
```

Provides QEMU user-mode networking using an emulated RTL8139 network card.

```text
-vnc 127.0.0.1:1
```

Starts the VNC server on display 1, which corresponds to TCP port **5901**.

---

## 14. Connect RealVNC Viewer to Windows 7

Keep the QEMU command running in Termux.

Open **RealVNC Viewer**.

Create a new connection.

Use this address:

```text
127.0.0.1::5901
```

The two colons before the port are intentional.

You are connecting to:

```text
Host: 127.0.0.1
Port: 5901
```

You should now see the QEMU display.

---

## 15. Install Windows 7

Inside the Windows 7 installer:

1. Select language.
2. Select keyboard/layout.
3. Select **Install now**.
4. Accept the Microsoft license terms.
5. Choose **Custom (advanced)**.
6. Select the virtual hard disk.
7. If the disk is shown as unallocated space, select it and continue.
8. Let Windows copy and expand the installation files.
9. QEMU will reboot several times.

### Very important during reboot

After Windows has copied the installation files, QEMU may reboot.

When the VM restarts, make sure it boots from the **virtual hard disk**, not from the ISO again.

If the installer starts from the beginning again, stop QEMU, remove the ISO from the command, and start the VM from the virtual disk.

---

## 16. First boot after Windows installation

Once Windows 7 finishes installation, you no longer need the ISO.

Start the VM without `-cdrom`:

```bash
cd ~/win7

qemu-system-x86_64 \
  -m 2048 \
  -smp 2 \
  -cpu core2duo \
  -machine pc \
  -vga std \
  -device usb-tablet \
  -drive file=win7.qcow2,format=qcow2,if=ide \
  -boot order=c \
  -nic user,model=rtl8139 \
  -vnc 127.0.0.1:1
```

Then connect RealVNC Viewer to:

```text
127.0.0.1::5901
```

---

## 17. Create a start script

This saves you from typing the long QEMU command every time.

Create the script:

```bash
cd ~/win7
nano start-win7.sh
```

Paste:

```bash
#!/data/data/com.termux/files/usr/bin/bash

VM_DIR="$HOME/win7"
DISK="$VM_DIR/win7.qcow2"

cd "$VM_DIR" || exit 1

termux-wake-lock

qemu-system-x86_64 \
  -m 2048 \
  -smp 2 \
  -cpu core2duo \
  -machine pc \
  -vga std \
  -device usb-tablet \
  -drive file="$DISK",format=qcow2,if=ide \
  -boot order=c \
  -nic user,model=rtl8139 \
  -vnc 127.0.0.1:1
```

Save in nano:

```text
CTRL + O
ENTER
CTRL + X
```

Make executable:

```bash
chmod +x start-win7.sh
```

Run Windows 7 with:

```bash
./start-win7.sh
```

Then open RealVNC Viewer and connect to:

```text
127.0.0.1::5901
```

---

## 18. Create a stop script

Create:

```bash
nano stop-win7.sh
```

Paste:

```bash
#!/data/data/com.termux/files/usr/bin/bash

pkill -f qemu-system-x86_64

termux-wake-unlock
```

Save and make executable:

```bash
chmod +x stop-win7.sh
```

Stop the VM:

```bash
./stop-win7.sh
```

### Safer shutdown

Inside Windows 7, use:

```text
Start → Shut down
```

Whenever possible, shut down Windows normally instead of force-killing QEMU.

---

## 19. Run the VM with more or less RAM

Default:

```bash
-m 2048
```

For a phone with plenty of RAM, you can test:

```bash
-m 3072
```

or:

```bash
-m 4096
```

Do not give almost all of the phone's RAM to Windows. Android and Termux still need memory.

For example, on a 6 GB phone, starting with:

```bash
-m 2048
```

is a reasonable experiment.

---

## 20. Change the virtual CPU count

Start with:

```bash
-smp 2
```

If stable, you can experiment with:

```bash
-smp 3
```

or:

```bash
-smp 4
```

More virtual CPUs do not automatically mean faster performance because the x86 CPU is being emulated on an ARM processor.

---

## 21. Improve Windows 7 usability

After Windows 7 starts:

### Disable unnecessary visual effects

In Windows 7:

```text
Control Panel
→ System
→ Advanced system settings
→ Advanced
→ Performance
→ Settings
→ Adjust for best performance
```

This can reduce the graphical workload.

### Use a lower Windows resolution

A lower resolution usually feels faster through VNC.

Try something such as:

```text
1024 × 768
```

or lower if necessary.

---

## 22. Check the VM files

List the VM:

```bash
ls -lh ~/win7
```

Typical result:

```text
Windows7.iso
win7.qcow2
start-win7.sh
stop-win7.sh
```

Check virtual disk information:

```bash
qemu-img info ~/win7/win7.qcow2
```

---

## 23. Check whether QEMU is running

Use:

```bash
ps -ef | grep qemu
```

Or:

```bash
pgrep -a qemu-system-x86_64
```

If QEMU is running, you should see the process.

---

## 24. Check whether VNC port 5901 is listening

Run:

```bash
ss -ltn | grep 5901
```

A listening entry means QEMU has opened the VNC port.

Because the command uses:

```text
-vnc 127.0.0.1:1
```

the VNC server is bound to the phone itself rather than exposed to other devices on your network.

---

## 25. RealVNC connection fails

Check these in order.

### A. Make sure QEMU is running

```bash
pgrep -a qemu-system-x86_64
```

### B. Check VNC

```bash
ss -ltn | grep 5901
```

### C. Use the correct RealVNC address

```text
127.0.0.1::5901
```

### D. Make sure QEMU uses the same display

Display 1:

```bash
-vnc 127.0.0.1:1
```

means port:

```text
5901
```

Display 0:

```bash
-vnc 127.0.0.1:0
```

would mean:

```text
5900
```

### E. Do not use your phone's Wi-Fi IP

For this same-phone setup, use:

```text
127.0.0.1
```

not something like:

```text
192.168.1.10
```

---

## 26. QEMU command not found

Check:

```bash
command -v qemu-system-x86_64
```

Then:

```bash
command -v qemu-system-x86-64
```

Check packages:

```bash
pkg list-installed | grep qemu
```

If QEMU is missing:

```bash
pkg update
pkg install qemu-system-x86-64 qemu-utils -y
```

---

## 27. `qemu-system-x86_64` crashes or library error appears

First update packages:

```bash
pkg update
pkg upgrade -y
```

Then reinstall the required QEMU packages:

```bash
pkg reinstall qemu-common qemu-utils qemu-system-x86-64
```

Run:

```bash
qemu-system-x86_64 --version
```

If Termux reports a missing library, do not randomly download `.so` files from the internet. Prefer fixing the Termux package dependencies through `pkg`.

---

## 28. QEMU installs but the x86_64 executable is missing

Check:

```bash
uname -m
dpkg --print-architecture
```

If you are running a 32-bit ARM Termux environment, current QEMU x86_64 packages may not provide the required emulator binary.

Do not continue by downloading random QEMU binaries.

Move to a supported 64-bit Termux/Android environment first.

---

## 29. Windows 7 freezes during boot

Try starting with less aggressive settings:

```bash
qemu-system-x86_64 \
  -m 2048 \
  -smp 1 \
  -cpu core2duo \
  -machine pc \
  -vga std \
  -device usb-tablet \
  -drive file=win7.qcow2,format=qcow2,if=ide \
  -boot order=c \
  -nic user,model=rtl8139 \
  -vnc 127.0.0.1:1
```

If that works, later test `-smp 2`.

---

## 30. Windows installer cannot see the disk

Make sure the disk option is present:

```bash
-drive file=win7.qcow2,format=qcow2,if=ide
```

For Windows 7 installation, the simple emulated IDE disk is preferable to using a storage controller that requires additional drivers.

---

## 31. Mouse is difficult to control

Keep:

```bash
-device usb-tablet
```

This gives the VM an absolute-position USB tablet device, which is useful with VNC.

If the mouse still behaves strangely, disconnect/reconnect the VNC session.

---

## 32. Windows 7 screen is black

Verify QEMU is started with:

```bash
-vga std
```

and:

```bash
-vnc 127.0.0.1:1
```

Then reconnect RealVNC to:

```text
127.0.0.1::5901
```

---

## 33. Android kills Termux while Windows is running

This is an Android background-process problem.

Before starting QEMU, run:

```bash
termux-wake-lock
```

The start script in this guide already does that.

Also review your phone's Android battery settings and set Termux to allow background activity / unrestricted battery usage where your Android version provides such an option.

Do not use aggressive memory-cleaner apps while the VM is running.

---

## 34. Phone gets very hot

QEMU x86 emulation can use a lot of CPU.

Try:

```bash
-m 2048
-smp 1
```

and reduce the Windows resolution.

Do not keep running the VM if the phone becomes excessively hot.

Using a charger can also increase heat.

---

## 35. Windows 7 is extremely slow

This is expected on many ARM phones.

The main reason is:

```text
ARM Android
      ↓
QEMU x86/x86_64 CPU emulation
      ↓
Windows 7
```

This is not the same as running an ARM operating system natively.

You can reduce the workload by:

- using 1024×768 or lower resolution
- disabling Windows visual effects
- using 1–2 virtual CPU cores
- closing other Android apps
- avoiding unnecessary Windows background services
- keeping the VM disk local and healthy

---

## 36. Network inside Windows 7

The example command uses:

```bash
-nic user,model=rtl8139
```

QEMU user networking generally allows the guest to make outbound network connections without needing the phone to act as a bridged Ethernet router.

Windows 7 must have a driver for the emulated adapter. If networking does not work, check:

```text
Control Panel
→ Device Manager
→ Network adapters
```

If the adapter is missing a driver, install the appropriate driver inside the VM only from a trustworthy source.

---

## 37. Do not expose the VNC server to the network unless you know what you are doing

This guide intentionally uses:

```bash
-vnc 127.0.0.1:1
```

That means localhost only.

Avoid changing it casually to:

```bash
-vnc :1
```

because that can make the VNC service reachable beyond localhost depending on the environment.

Basic VNC password authentication is not strong modern security, so keeping it localhost-only is important.

---

## 38. Optional: VNC password

For a same-phone localhost setup, you normally do not need a VNC password.

QEMU also supports password authentication, but classic VNC password authentication has security limitations. Do not treat an old VNC password as strong protection.

If you experiment with VNC password authentication, keep the server bound to:

```text
127.0.0.1
```

rather than exposing it to the network.

---

## 39. Normal daily workflow

### Start Windows 7

Open Termux:

```bash
cd ~/win7
./start-win7.sh
```

Then open RealVNC Viewer:

```text
127.0.0.1::5901
```

### Use Windows 7

Operate Windows 7 through the RealVNC screen.

### Shut down Windows

Inside Windows:

```text
Start → Shut down
```

Then return to Termux.

If QEMU is still running after Windows shuts down:

```bash
./stop-win7.sh
```

---

## 40. Quick start commands

After everything is installed, these are the commands you normally need:

```bash
cd ~/win7
./start-win7.sh
```

Then connect RealVNC to:

```text
127.0.0.1::5901
```

To check QEMU:

```bash
pgrep -a qemu-system-x86_64
```

To check VNC:

```bash
ss -ltn | grep 5901
```

To stop:

```bash
./stop-win7.sh
```

---

## 41. Complete folder layout

Recommended layout:

```text
~/win7/
├── Windows7.iso
├── win7.qcow2
├── start-win7.sh
└── stop-win7.sh
```

---

## 42. Backup the Windows 7 VM

The most important file is:

```text
win7.qcow2
```

It contains your Windows installation and data.

You can copy it to another storage location while the VM is **completely powered off**.

Example:

```bash
cp ~/win7/win7.qcow2 ~/storage/shared/
```

Do not copy a changing VM disk as if it were a normal static file.

---

## 43. Restore the VM

If you have a backup:

```text
win7.qcow2
```

put it back into:

```text
~/win7/
```

Then start:

```bash
cd ~/win7
./start-win7.sh
```

You do not need to reinstall Windows if the QCOW2 disk already contains the installed VM.

---

## 44. Important limitations

### Performance

Expect slow performance compared with a normal computer because x86/x86_64 is being emulated on an ARM phone.

### Hardware acceleration

Do not assume KVM acceleration is available for x86 Windows guests on your Android phone. On most ordinary ARM phones, the practical fallback is QEMU TCG software emulation.

### Graphics

VNC is convenient but not ideal for high-performance graphics.

### Windows support

Windows 7 is obsolete and unsupported. Use it only for appropriate legacy/testing purposes.

### Battery

A running VM can consume substantial battery even while connected to a charger.

### Heat

Long VM sessions can make the phone hot.

---

## 45. One-command diagnostic collection

When something does not work, collect these outputs:

```bash
echo "=== Architecture ==="
uname -m
dpkg --print-architecture

echo "=== Android/Termux ==="
termux-info

echo "=== QEMU ==="
command -v qemu-system-x86_64
qemu-system-x86_64 --version

echo "=== QEMU packages ==="
pkg list-installed | grep qemu

echo "=== VM files ==="
ls -lh ~/win7

echo "=== Disk info ==="
qemu-img info ~/win7/win7.qcow2

echo "=== QEMU process ==="
pgrep -a qemu-system-x86_64

echo "=== VNC port ==="
ss -ltn | grep 5901
```

Save or copy the output if you need troubleshooting.

---

## 46. Sources / further reading

Termux official project:

https://github.com/termux/termux-app

QEMU documentation:

https://www.qemu.org/docs/master/

QEMU system emulator documentation:

https://www.qemu.org/docs/master/system/

QEMU VNC documentation:

https://www.qemu.org/docs/master/tools/qemu-vnc.html

QEMU VNC security documentation:

https://www.qemu.org/docs/master/system/vnc-security.html

RealVNC Viewer / Connect documentation:

https://help.realvnc.com/

---

## 47. Final checklist

Before starting:

```text
[ ] Termux installed from an official source
[ ] Termux packages updated
[ ] Storage permission granted
[ ] Phone is 64-bit ARM / aarch64
[ ] QEMU x86_64 works
[ ] Windows 7 ISO is available
[ ] win7.qcow2 exists
[ ] RealVNC Viewer is installed
```

Start:

```bash
cd ~/win7
./start-win7.sh
```

Connect:

```text
127.0.0.1::5901
```

Stop safely:

```text
Windows → Start → Shut down
```

Then:

```bash
./stop-win7.sh
```

---

## Note about commands

Termux package names and QEMU package layouts can change. If an exact package name in this README does not exist on your current Termux installation, first run:

```bash
pkg search qemu
```

and:

```bash
pkg list-installed | grep qemu
```

Use the QEMU executable actually provided by your installed package.

---

## 👨‍💻 Author

**Raju Prajapat**

Electronics & Communication Engineering Student.

**Connect**

- GitHub: [https://github.com/raju-prajapat](https://github.com/raju-prajapat)
- LinkedIn: [https://www.linkedin.com/in/raju-prajapat-839b49394](https://www.linkedin.com/in/raju-prajapat-839b49394)

---

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for more information.

---

## ⭐ Support

If you find this project useful, consider giving the repository a ⭐ on GitHub.

