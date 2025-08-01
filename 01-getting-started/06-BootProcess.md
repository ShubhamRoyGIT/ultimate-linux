**Linux Boot Process**

The Linux boot process is the sequence of steps your system goes through to load the operating system — from powering on to the login prompt or GUI.

🧠 Overview: 6 Stages of the Linux Boot Process
Step	Component	What It Does
- 1️⃣	"BIOS / UEFI"	Initializes hardware and finds bootable media
- 2️⃣	"Bootloader (GRUB)"	Loads kernel and initial RAM disk
- 3️⃣	"Kernel"	Loads drivers, mounts root filesystem
- 4️⃣	"initramfs"	Temporary root to help mount real root FS
- 5️⃣	"Init system (systemd, SysV)"	Starts system services (network, SSH, etc.)
- 6️⃣	"Login prompt / GUI"	User logs in via terminal or desktop

🔍 Step-by-Step Breakdown


🏁 1. BIOS(Old System) / UEFI(Modern System) - (Program Stored in Seperate Chip on Motherboard)
- Performs POST (Power-On Self Test) - check if all hardware components have power
- Detects CPU, RAM, disks, keyboard, etc.
- Starts scanning the bootable device listed in BIOS Settings device list
- Loads the bootloader from the MBR or EFI partition

  
  a. 🔧 BIOS = legacy, 512-byte MBR = BIOS Reads first 512 bytes of MBR partition

  
  b. 🔧 UEFI = modern, supports larger disks and secure boot

  
- Booatloader is loaded


🚀 2. Bootloader (usually GRUB)

GRUB (GRand Unified Bootloader) is the most common

- Displays boot menu (e.g., Ubuntu, Recovery)
- Loads the Linux kernel and initramfs into memory

```bash
/boot/grub/grub.cfg
```

🧩 3. Linux Kernel


Uncompressed and initialized by GRUB

- Initializes CPU, memory, basic hardware
- Mounts a temporary filesystem using initramfs
- Starts PID 1 (initial process)

At this point, you have a running kernel — but not yet a usable system.


📦 4. initramfs (Initial RAM Filesystem)


- A temporary root filesystem (in RAM)
- It’s a compressed cpio archive embedded into the Linux kernel image or loaded alongside it by the bootloader (e.g., GRUB).
- It contains a minimal root filesystem and essential binaries (like sh, mount, ls, udev, etc.).
- Its job is to prepare the environment until the actual root filesystem (like /dev/sda1) is ready to be mounted.


📦 How Linux Mounts a Temporary Filesystem Using initramfs

🔁 Step-by-Step Flow:


- Kernel loads initramfs into memory
- Kernel unpacks the archive into a tmpfs filesystem (temporary memory-backed filesystem).
- Mounted at / (yes, it’s the first root filesystem).
- This is an in-memory root (RAM-based), not your final disk-based root.
- /init script inside initramfs runs
- This is the entry point of the temporary user space.
- Typically a shell script or binary (like init, systemd, or custom logic).

Prepares for real root:

- Loads drivers/modules (e.g., for LVM, RAID, encrypted disks)
- Detects block devices
- Mounts the actual root filesystem (e.g., /dev/sda2)
- Switches to the real root filesystem
- Uses switch_root or pivot_root to replace the current root (/) with the real one.
- Mounts /proc, /sys, and /dev again inside the new root.

Transfers control to PID 1 (systemd/init)
Continues the rest of the Linux boot process from there.

📍 Where is this temporary root mounted?
Initially: initramfs is mounted at /

Later: the real root filesystem replaces it

initramfs is discarded after switching to the real root (releases memory)

🔍 Inspecting It
You can view what your system used during boot:
```bash
dmesg | grep initramfs     #Or, check how the switch happened:
journalctl -b | grep switch_root


📌 Example: Contents of an initramfs
You can extract an initramfs image like this:


mkdir /tmp/initramfs
cd /tmp/initramfs
zcat /boot/initrd.img-$(uname -r) | cpio -idmv
Inside, you'll find:
/init
/bin/
/lib/
/dev/
/proc/
/sys/
This is your early user space.
```


Example: Loads RAID/LVM/crypto drivers if needed

Then switches to the real root filesystem

🔧 5. Init System (systemd, init, upstart)


- Controls what services and daemons start
- Responsible for the boot target (multi-user, graphical)
- On most modern systems: systemd
```bash
# First user-space process
/sbin/init  ➝ systemd (PID 1)
Starts things like:

Network manager
SSH
Display manager
Logging
Custom services

```

👤 6. Login (CLI or GUI)


You’re presented with a:
- Terminal login prompt (getty) on server
- GUI login manager (gdm, lightdm) on desktop
- System is now fully booted and operational

  

🧭 Visual Summary


[ BIOS / UEFI ]


       ↓

       
[ GRUB Bootloader ]

       ↓

       
[ Linux Kernel ]

       ↓

       
[ initramfs ]

       ↓

       
[ /sbin/init → systemd ]

       ↓

       
[ User Login Prompt ]




🔎 Want to Inspect Boot on a Live System?
```bash
dmesg | less        # Kernel messages
journalctl -b       # Systemd boot logs
systemd-analyze     # Boot time analysis
```
