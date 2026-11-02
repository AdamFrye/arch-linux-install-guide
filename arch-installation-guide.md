# Arch Linux Installation Guide

## Pre-Installation

- Downloaded the Arch ISO from MIT.
- Allocated **8 GB of RAM**, **32 GB of storage**, and **4 cores**; set **Enable EFI** to `true`.
- Once logged into the live environment with a `zsh` shell.
- Ran `ip link` to verify access to the ethernet (it showed that `enp0s3` was `UP`).
- `ping ping.archlinux.org` returned packets, verifying the internet connection.
- Set the clock timezone to my local region, `America/Chicago`, using:

  ```
  timedatectl set-timezone America/Chicago
  ```

## Partitioning the Disks

1. See the available devices:

   ```
   fdisk -l
   ```

2. Open the `fdisk` CLI (in my case):

   ```
   fdisk /dev/sda
   ```

3. Type `g` at the prompt to initialize a **GPT** disk label.
4. Type `n` to begin creating a new partition.
5. Set the first sector to `2048` and the last sector to `+1G` — a **1 GB** partition for `/boot`.
6. Repeat for a **4 GB** partition for **swap**.
7. Repeat again for `/` using the remaining space on the drive (allocate all remaining space for the last sector).
8. Type `w` to write the changes and exit the CLI.

## Formatting the Partitions



| Partition  | Command |
|------------|---------|
| `/boot`    | `mkfs.vfat -F 32 /dev/<partition>` |
| `/` (root) | `mkfs.ext4 /dev/<partition>` |
| swap       | `mkswap /dev/<partition>` |

## Mounting the File Systems

- Mount the root file system at `/mnt`:

  ```
  mount /dev/sda3 /mnt
  ```

- Mount boot under `/mnt/boot` (no need to mount swap):

  ```
  mkdir /mnt/boot && mount /dev/sda1 /mnt/boot
  ```

- Enable swap space:

  ```
  swapon /dev/sda2
  ```

## Installation

- Install the base packages required to run the distro:

  ```
  pacstrap -K /mnt base linux linux-firmware
  ```

- Generate the `fstab` file:

  ```
  genfstab -U /mnt >> /mnt/etc/fstab
  ```

- Jump into the new environment (press `Ctrl` + `]` to exit):

  ```
  arch-chroot -S /mnt
  ```

- Set the time:

  ```
  ln -sf /usr/share/zoneinfo/Area/Location /etc/localtime
  hwclock --systohc
  ```

- Set the locale:

  ```
  local-gen && echo "LANG=en_US.UTF-8" > /etc/locale.conf
  ```

- Set the hostname:

  ```
  echo "arch-vm" > /etc/hostname
  ```

- Also run (whatever this is):

  ```
  mkinitcpio -P
  ```

- Change the root password:

  ```
  passwd
  ```

## Boot Loader

- Download GRUB and `efibootmgr` (including `linux` just in case):

  ```
  pacman -S grub efibootmgr linux dhcpcd
  ```

- Install GRUB:

  ```
  grub-install --target=x86_64-efi --efi-directory=/boot --boot-directory=/boot
  ```

- Generate the GRUB config:

  ```
  grub-mkconfig -o /boot/grub/grub.cfg
  ```

## Misc

- Added my user to the `wheel` group:

  ```
  useradd -m -G wheel adam
  ```

- Set a password:

  ```
  passwd adam
  ```

- Set up the network:

  ```
  systemctl enable dhcpcd
  ```

## Reboot

- Exit the chroot environment:

  ```
  exit
  ```

- Unmount all of the drives:

  ```
  umount -R /mnt
  ```

- Reboot the virtual machine:

  ```
  reboot
  ```

## In Arch

- Add `codi` to the `wheel` group:

  ```
  useradd -m -G wheel codi
  ```

- Uncomment the line in `sudoers` that allows `wheel` group members to run `sudo` commands.
- Disable `dhcpcd`:

  ```
  systemctl disable --now dhcpcd
  ```

- Download and enable NetworkManager:

  ```
  pacman -S networkmanager && systemctl enable --now NetworkManager
  ```

## Installing a Desktop Environment

- Install KDE Plasma and `sddm`:

  ```
  sudo pacman -S plasma-meta sddm
  ```

- Enable on startup:

  ```
  sudo systemctl enable sddm
  ```

- Reboot:

  ```
  reboot
  ```

## Coloring the Terminal

- Open `nano ~/.bashrc`
- Append:

  ```
  PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
  ```

## Problems I Ran Into

### GRUB couldn't boot (BIOS legacy mode)

- **Problem:** I initialized the disk label as GPT, but since I was in BIOS legacy mode, GRUB couldn't work.
- **Solution:** I turned the VM into UEFI mode through the VirtualBox GUI.

### GRUB couldn't find the Linux instance

- **Problem:** GRUB couldn't find the Linux instance.
- **Solution:** I ran `pacman -S linux` and then regenerated the GRUB config.
