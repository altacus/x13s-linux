# x13s-linux

Linux kernel for the Lenovo ThinkPad X13s (SC8280XP) with **Venus video acceleration**.

This is a fork of [gregkh/linux](https://github.com/gregkh/linux) (the stable kernel tree) with a handful of patches on top of **v7.0.10** to enable hardware video encode/decode and other X13s-specific configuration.

## Branches

| Branch | Base | What's there |
|--------|------|-------------|
| `main` | v7.0.10 | v7.0.10 + Venus patches + johan_defconfig |

The Venus video patches were cherry-picked from [jhovold/linux](https://github.com/jhovold/linux) (wip/sc8280xp-6.16 branch), which is the upstream reference for X13s kernel development.

## Building

```bash
git clone https://github.com/altacus/x13s-linux.git
cd x13s-linux
make johan_defconfig
make -j$(nproc) bindeb-pkg
```

Install the resulting `.deb` packages:

```bash
sudo dpkg -i ../linux-image-*.deb ../linux-headers-*.deb ../linux-libc-dev_*.deb
sudo cp arch/arm64/boot/dts/qcom/sc8280xp-lenovo-thinkpad-x13s.dtb /boot/
sudo update-grub
```

> **Note:** The DTB must be manually copied to `/boot/` to ensure GRUB loads the correct one. Configure `GRUB_DTB` in `/etc/default/grub` if needed.

## What works

- Wi-Fi (ath11k)
- Bluetooth (wcn6855) — a systemd service to set the public MAC address on boot is recommended
- Audio (speakers, mic, headset via SC8280XP sound card)
- Display panel and GPU (Adreno, DRM)
- USB-C and DP alt mode
- NVMe storage
- UFS storage
- Camera (libcamera)
- Microphone recording
- Venus hardware video encode/decode
- All 3 remote processors: SLPI (sensor), ADSP (audio), CDSP (compute)
- fscrypt (AES-256-XTS and AES-256-CBC-CTS via ARM CE)
- Suspend/resume
- Power management / cpufreq (schedutil governor)

## Kernel config

The `johan_defconfig` (from Johan Hovold) is a well-tuned config for the X13s. Additional options added on top:

- `CONFIG_FS_ENCRYPTION` (fscrypt)
- `CONFIG_NF_TABLES` and all nftables modules
- `CONFIG_MD_BITMAP_FILE`
- `CONFIG_CRYPTO_SHA512_ARM64`
- `CONFIG_PCIE_QCOM=y` (built-in, not module)

## dmesg notes

Some messages are normal for this platform and can be ignored:

- `PSCI Firmware Bug: No CPU operations for boot mode PC` — firmware quirk
- `phy clock deferrals (-517)` — probe ordering, resolves on retry
- `pmic-glink device link` warnings — cosmetic

## Credits

- [Johan Hovold](https://github.com/jhovold) — X13s kernel bring-up and the upstream wip branches
- [Konrad Dybcio](https://github.com/konradybcio) — Venus SC8280XP/SM8350 enablement patches
- [Greg Kroah-Hartman](https://github.com/gregkh) — stable kernel tree
