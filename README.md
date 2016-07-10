# MK808B
Unofficial MK808B Android/Linux support

### 1) Building a linux boot image for dual boot

**Install dependencies:**
```
sudo apt-get install build-essential lzop libncurses5-dev libssl-dev libc6:i386 lib32z1 lib32stdc++6
```

**Install the toolchain:**
```
mkdir toolchain
cd toolchain
wget https://releases.linaro.org/13.04/components/toolchain/binaries/gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.bz2
tar xvf gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.bz2
rm gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.bz2
toolchain="$(pwd)/gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux/bin"
cat <<EOF >>  ~/.bashrc
export PATH="$toolchain:$PATH"
EOF
source ~/.bashrc
```

**Clone kernel sources:**
```
git clone https://github.com/BalintBanyasz/kernel_rockchip.git --branch rockchip-3.0-rbox-kk --single-branch rockchip-3.0-rbox-kk
```

**Generate initrd image:**
```
git https://github.com/BalintBanyasz/initrd.git
make -C initrd
```

The generated initrd.img image file will be located in the current folder.

Gzip the initrd.img file to get a compressed boot image:
```
gzip -c initrd.img > initrd.cpio
```

The initramfs image will be embedded into the kernel.

**Build the kernel:**
```
cd rockchip-3.0-rbox-kk
make mrproper
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- rk30_mk808b_linux_defconfig
make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
cd ..
 ```
 
**Generate boot (recovery) image:**

First, generate a fakeramdisk.gz:
```
dd if=/dev/zero of=fakeramdisk count=1M bs=1
gzip -9 fakeramdisk
```
Now, create a boot image:
```
mkbootimg --kernel rockchip-3.0-rbox-kk/arch/arm/boot/Image --ramdisk fakeramdisk.gz --base 60400000 --pagesize 16384 --ramdiskaddr 62000000 -o recovery.img
```

Flash recovery.img to the recovery partition of your device to be able to boot a Linux system from your SD-card.
