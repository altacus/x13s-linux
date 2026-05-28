# DTB Management on the ThinkPad X13s

The X13s uses a flattened device tree (DTB) passed to the kernel by the bootloader (GRUB). Keeping the DTB in sync with the kernel is critical — an out-of-date DTB can cause missing devices, probe failures, or boot hangs.

## How it works

1. The kernel build produces `arch/arm64/boot/dts/qcom/sc8280xp-lenovo-thinkpad-x13s.dtb`
2. Debian package builds (`make bindeb-pkg`) install it to `/boot/dtbs/<kernel-version>/qcom/`
3. GRUB loads the DTB specified by `GRUB_DTB` in `/etc/default/grub`

## The problem

`update-grub` creates a symlink `/boot/dtbs/current → /boot/dtbs/<kernel-version>`, but `GRUB_DTB` points to an **explicit path** (`/sc8280xp-lenovo-thinkpad-x13s.dtb` under `/boot/`). After a kernel update, the old DTB remains at that path unless manually replaced — resulting in a mismatch between the running kernel and its DTB.

## Verifying the loaded DTB

Check which DTB the kernel actually loaded:

```bash
cat /sys/firmware/devicetree/base/model
# Should show: "Lenovo ThinkPad X13s"
dmesg | grep "DTB"
# Shows the DTB memory address if loaded
```

If Venus or other devices are missing from `lspci` / `ls /dev/video*`, the wrong DTB is likely loaded.

## Recommended workflow after a kernel build

```bash
# Build kernel packages
make -j$(nproc) bindeb-pkg

# Install
sudo dpkg -i ../linux-image-*.deb ../linux-headers-*.deb

# Copy the new DTB to /boot
sudo cp arch/arm64/boot/dts/qcom/sc8280xp-lenovo-thinkpad-x13s.dtb /boot/

# Update GRUB
sudo update-grub

# Reboot
sudo reboot
```

## GRUB configuration

In `/etc/default/grub`, ensure these lines are set:

```
GRUB_DEFAULT_DTB="sc8280xp-lenovo-thinkpad-x13s.dtb"
GRUB_DTB="/sc8280xp-lenovo-thinkpad-x13s.dtb"
```

The DTB file must be at `/boot/sc8280xp-lenovo-thinkpad-x13s.dtb` (GRUB reads from `/boot/` regardless of the kernel version).

## Automation

To avoid forgetting, create a post-install hook or alias:

```bash
# ~/.bash_aliases
alias kernel-install='sudo cp arch/arm64/boot/dts/qcom/sc8280xp-lenovo-thinkpad-x13s.dtb /boot/ && sudo update-grub'
```

Or use a pacman/postinst hook if you build Debian packages regularly.

## Verifying the DTB matches your kernel

The kernel and DTB must be from the **same build**. You can check:

```bash
# Find the kernel build timestamp
cat /proc/version
# Then check if the DTB was built at the same time (stat the source .dts)
```

## Recovering from a bad DTB

If you copy a DTB that causes boot failure:

1. Reboot and hold Shift (or spam Esc) to enter GRUB menu
2. Press `e` to edit the boot entry
3. Remove the `devicetree` line (or fix the path)
4. Boot with the fallback initramfs DTB
5. Copy back the working DTB once booted
