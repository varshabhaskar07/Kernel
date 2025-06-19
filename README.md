
# Setting Up Artix OS VM and Compiling Custom Linux Kernel

## 📌 Objective
Set up a virtual machine running **Artix Linux (runit)** using **text-based installation**, configure it with required developer tools, **compile the Linux kernel from source**, and boot into the custom-built kernel.

---

## 🖥️ 1. Virtual Machine Setup

### 💡 VM Configuration
| Setting        | Value            |
|----------------|------------------|
| RAM            | 4 GB             |
| vCPUs          | 2                |
| Disk Space     | 20 GB            |
| Boot Mode      | UEFI             |
| Sound/USB      | Disabled (optional) |
| Platform       | VirtualBox / VMware / QEMU |

### 📥 ISO Download
Download the **Artix Base ISO (runit version)** from the official site:  
👉 [https://artixlinux.org/download.php](https://artixlinux.org/download.php)

---

## 🧰 2. Artix Linux Installation (Text Mode)

### 🧱 Partitioning Example
Using `cfdisk`, `fdisk`, or `parted`:
- `/dev/sda1` – EFI System Partition (512 MB, FAT32)
- `/dev/sda2` – Root partition (`/`, ext4)

### 🔧 Mounting
```bash
mount /dev/sda2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
````

### 📦 Install Base System

```bash
basestrap /mnt base base-devel runit elogind-runit linux linux-firmware
```

### 📁 File System Configuration

```bash
fstabgen -U /mnt >> /mnt/etc/fstab
```

### 🚪 Chroot into New System

```bash
artix-chroot /mnt
```

### 🌐 System Setup

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
echo myhostname > /etc/hostname
nano /etc/locale.gen   # Uncomment en_US.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

### 🧷 Bootloader (GRUB)

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Artix
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 📦 3. Post-Installation Tasks

### ✅ Enable Arch Repositories

Edit `/etc/pacman.conf`:

```ini
[extra]
Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist
```

```bash
sudo pacman -Sy
```

### 📦 Install Required Packages

```bash
sudo pacman -S binutils elfutils gcc gdb make automake autoconf yasm vim
```

---

## 🐧 4. Kernel Compilation

### 🔽 Download & Extract Kernel Source

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.x.x.tar.xz
tar -xf linux-6.x.x.tar.xz
cd linux-6.x.x
```

### 🔃 Prepare for Build

```bash
make mrproper
```

### 📋 Use Provided Kernel Config

Assuming file is `config-rev-9-gold`:

```bash
cp /path/to/config-rev-9-gold .config
make nconfig    # Press ESC to exit and update automatically
```

### 🛠️ Compile the Kernel

```bash
make -j$(nproc)
```

### 📥 Install Kernel and Modules

```bash
sudo make modules_install
sudo make install
```

This generates:

* `/boot/vmlinuz-*`
* `/boot/initramfs-*`

---

## 🔃 5. Update GRUB and Reboot

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

From the GRUB menu, select the **new kernel version**.

---

## ✅ Verification Checklist

| Task                                       | Status |
| ------------------------------------------ | ------ |
| VM boots with UEFI                         | ✅      |
| Partitions (EFI + root) created properly   | ✅      |
| Artix base (runit) installed via text mode | ✅      |
| ArchLinux repos enabled                    | ✅      |
| Required packages installed                | ✅      |
| Kernel compiled using provided config      | ✅      |
| VM successfully boots into compiled kernel | ✅      |


---

## 🏁 Conclusion

Successfully set up a lean Artix Linux virtual machine, installed essential developer tools, compiled the Linux kernel from source using a custom config, and verified the successful boot.
