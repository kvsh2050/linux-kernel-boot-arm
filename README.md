<div align="center">
  <h1>Linux Kernel Scratch</h1>
  
  <p>
    Building and booting a Linux Kernel from scratch on ARM64 with BusyBox
  </p>
  
<!-- Badges -->
<p>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/graphs/contributors">
    <img src="https://img.shields.io/github/contributors/kvsh2050/linux-kernel-scratch" alt="contributors" />
  </a>
  <a href="">
    <img src="https://img.shields.io/github/last-commit/kvsh2050/linux-kernel-scratch" alt="last update" />
  </a>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/network/members">
    <img src="https://img.shields.io/github/forks/kvsh2050/linux-kernel-scratch" alt="forks" />
  </a>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/stargazers">
    <img src="https://img.shields.io/github/stars/kvsh2050/linux-kernel-scratch" alt="stars" />
  </a>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/issues/">
    <img src="https://img.shields.io/github/issues/kvsh2050/linux-kernel-scratch" alt="open issues" />
  </a>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/blob/master/LICENSE">
    <img src="https://img.shields.io/github/license/kvsh2050/linux-kernel-scratch.svg" alt="license" />
  </a>
</p>
   
<h4>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch">View Demo</a>
  <span> · </span>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch">Documentation</a>
  <span> · </span>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/issues/">Report Bug</a>
  <span> · </span>
  <a href="https://github.com/kvsh2050/linux-kernel-scratch/issues/">Request Feature</a>
</h4>
</div>

<br />

<!-- Table of Contents -->
# :notebook_with_decorative_cover: Table of Contents

- [About the Project](#star2-about-the-project)
  * [Features](#dart-features)
- [Getting Started](#toolbox-getting-started)
  * [Prerequisites](#bangbang-prerequisites)
  * [Build the Kernel](#gear-build-the-kernel)
  * [Build BusyBox Root FS](#package-build-busybox-root-fs)
  * [Create Initramfs](#wrench-create-initramfs)
  * [Run in QEMU](#rocket-run-in-qemu)
  * [Extract Device Tree](#deciduous_tree-extract-device-tree-optional)
- [Contact](#handshake-contact)
- [Acknowledgements](#gem-acknowledgements)


<!-- About the Project -->
## :star2: About the Project

A step-by-step guide and build scripts for compiling a **Linux Kernel v6.6** from source for **ARM64 (Cortex-A)**, building a minimal **BusyBox** root filesystem, packaging it as an **initramfs**, and booting the whole thing inside **QEMU** — no physical hardware required.

### :dart: Features

- Cross-compiled Linux v6.6 kernel targeting `aarch64`
- Minimal BusyBox root filesystem with a custom `init` script
- Boots to an interactive shell via QEMU `virt` machine
- Optional device tree (DTB/DTS) extraction from QEMU
- Fully reproducible — no distro-specific tooling beyond the listed dependencies


<!-- Getting Started -->
## :toolbox: Getting Started

### :bangbang: Prerequisites

Install all required build tools and the QEMU AArch64 emulator:

```bash
sudo apt update -y && sudo apt install -y \
  zsh git gcc flex bison make ca-certificates curl \
  gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu \
  libc6-dev-arm64-cross qemu-system-aarch64 qemu-user \
  qemu-user-static bc libssl-dev libncurses-dev \
  libncurses5-dev libncursesw5-dev bzip2 vim nano cpio \
  device-tree-compiler
```

---

### :gear: Build the Linux Kernel (v6.6)

```bash
git clone --depth 1 --branch v6.6 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
make ARCH=arm64 defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) Image dtbs
cd ..
```

---

### :package: Build BusyBox (Root FS)

```bash
git clone --depth=1 https://git.busybox.net/busybox.git
cd busybox
make ARCH=arm64 defconfig
# Optional: set CONFIG_STATIC=y in .config for a standalone initramfs
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- install
```

---

### :wrench: Create Initramfs

```bash
cd _install

cat > init <<'EOF'
#!/bin/sh
# Simple init: print a message, drop to a shell
echo "Hello, welcome to the Linux kernel — this is initramfs!" > /dev/console
export PATH=/bin:/sbin:/usr/bin:/usr/sbin
exec /bin/sh
EOF

chmod +x init
find . -print0 | cpio --null -ov --format=newc > ../../initramfs.cpio
cd ../..
```

---

### :rocket: Run in QEMU

```bash
qemu-system-aarch64 \
  -machine virt \
  -cpu cortex-a57 \
  -nographic \
  -kernel linux/arch/arm64/boot/Image \
  -initrd initramfs.cpio
```

You should see the boot log followed by the `initramfs` welcome message and an interactive shell.

---

### :deciduous_tree: Extract Device Tree (Optional)

Dump and decompile the QEMU `virt` machine's device tree:

```bash
qemu-system-aarch64 -machine virt -machine dumpdtb=qemu.dtb
dtc -I dtb qemu.dtb > qemu.dts
```


<!-- Contact -->
## :handshake: Contact

Kavya — [@kvsh2050](https://github.com/kvsh2050) — email@email_client.com

Project Link: [https://github.com/kvsh2050/linux-kernel-scratch](https://github.com/kvsh2050/linux-kernel-scratch)


<!-- Acknowledgements -->
## :gem: Acknowledgements

- [The Linux Kernel](https://www.kernel.org/)
- [BusyBox](https://www.busybox.net/)
- [QEMU](https://www.qemu.org/)
- [Shields.io](https://shields.io/)
- [Awesome README](https://github.com/matiassingers/awesome-readme)
- [Readme Template](https://github.com/othneildrew/Best-README-Template)
