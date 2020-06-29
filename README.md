# Thor96

LPDDR4, Micron:
MT53B512M32D2NP-062 WT:D
2-GB LPDDR4 @ 1,600 MHz Industrial Temp by Micron
http://files.pine64.org/doc/datasheet/rockpro64/SM512M32Z01MD2BNP(200BALL).pdf

Execute commands on a ARM64 platform (maybe directly on iMX8MQ Thor96)


## Binaries from NXP site

Files are needed for compilation of U-Boot

```
# Fetch ddr and hdmi binary blobs

mkdir -p u-boot
wget https://www.nxp.com/lgfiles/NMG/MAD/YOCTO/firmware-imx-8.7.bin
chmod +x firmware-imx-7.9.bin
./firmware-imx-7.9.bin
```


## ATF

```
git clone https://source.codeaurora.org/external/imx/imx-atf
cd imx-atf
git checkout d6451cc1e
make PLAT=imx8mq bl31
cd ..
```


## Patches for U-Boot and Kernel from Einfochips

```
git clone https://github.com/iohe/thor96.git
```

## U-Boot

```
git clone https://source.codeaurora.org/external/imx/uboot-imx
cd uboot-imx
git checkout imx_v2018.03_4.14.78_1.0.0_ga
cp ../thor96/uboot-imx/0001-Add-Thor96-original-changes.patch .
git apply 0001-Add-Thor96-original-changes.patch
make imx8mq_thor96_defconfig
make
cd ..
```

## flash.bin

```
git clone https://source.codeaurora.org/external/imx/imx-mkimage
cd imx-mkimage
git checkout dd02340
cd ..

cd uboot-imx
cp tools/mkimage ../imx-mkimage/iMX8M/mkimage_uboot
cp spl/u-boot-spl.bin  ../imx-mkimage/iMX8M
cp u-boot-nodtb.bin ../imx-mkimage/iMX8M
cp arch/arm/dts/fsl-imx8mq-thor96.dtb ../imx-mkimage/iMX8M/fsl-imx8mq-evk.dtb

cd ../imx-atf
cp build/imx8mq/release/bl31.bin ../imx-mkimage/iMX8M/
cd ..

cp firmware-imx-7.9/firmware/hdmi/cadence/signed_hdmi_imx8m.bin imx-mkimage/iMX8M
cp firmware-imx-7.9/firmware/ddr/synopsys/lpddr4_pmu_train* imx-mkimage/iMX8M

cd imx-mkimage
make SOC=iMX8M flash_hdmi_spl_uboot

# /dev/sdb is in this case the sd card

sudo dd if=iMX8M/flash.bin of=/dev/sdb bs=1024 seek=33
cd ..
```

## Linux

```
git clone https://source.codeaurora.org/external/imx/linux-imx.git
cd linux-imx
# git checkout imx_4.14.78_1.0.0_ga = 94da7bdc489b
git checkout 94da7bdc489b
cp ../thor96/linux-imx/0001-Thor96-Einfochips.patch .
git apply 0001-Thor96-Einfochips.patch


# Binary blobs that i choose to bake into kernel (sdma and brcmfmac(Murata 1MW)).
mkdir -p firmware/imx/sdma
cp ../firmware-imx-7.9/firmware/sdma/sdma-imx7d.bin firmware/imx/sdma/

# firmware/brcm/brcmfmac43455-sdio.txt
# firmware/brcm/brcmfmac43455-sdio.bin
# firmware/brcm/brcmfmac43455-sdio.clmblob
# Excelent info here: https://community.nxp.com/thread/491236
#[    4.428654] brcmfmac: brcmf_fw_map_chip_to_name: using brcm/brcmfmac43455-sdio.bin for chip 0x004345(17221) rev 0x000006
#[    4.581798] brcmfmac: brcmf_c_preinit_dcmds: Firmware version = wl0: Feb  7 2018 02:58:50 version 7.45.151 (r683645 CY) FWID 01-baba21a4
d
#
mkdir -p firmware/brcm
cp ../thor96/linux-imx/firmware/brcm/* firmware/brcm

mkdir -p build_evk
make -j4 LOADADDR=20008000 O=build_evk defconfig
cp ../thor96/linux-imx/build_evk/.config build_evk/.config
make -j4 LOADADDR=20008000 O=build_evk

# copy build_evk/arch/arm64/boot/Image and 
#   build_evk/arch/arm64/boot/dts/freescale/fsl-imx8mq-thor96.dtb first partition of sd-card.
```


