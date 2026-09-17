# redmi-ax6000-immortalwrt-hanwckf

从 [immortalwrt-mt798x](https://github.com/hanwckf/immortalwrt-mt798x) 项目编译而来。[原作者的介绍](https://cmi.hanwckf.top/p/immortalwrt-mt798x/)。

该仓库中 Redmi AX6000 的固件有[两个变种](https://github.com/hanwckf/immortalwrt-mt798x/blob/ba554197ed7e252fd3c1ee023621e1d1d009d323/target/linux/mediatek/image/mt7986.mk#L367-L397)

- [原厂分区表 (stock layout)](https://github.com/hanwckf/immortalwrt-mt798x/blob/ba554197ed7e252fd3c1ee023621e1d1d009d323/target/linux/mediatek/files-5.4/arch/arm64/boot/dts/mediatek/mt7986a-xiaomi-redmi-router-ax6000-stock.dts)
- [110m 分区表](https://github.com/hanwckf/immortalwrt-mt798x/blob/ba554197ed7e252fd3c1ee023621e1d1d009d323/target/linux/mediatek/files-5.4/arch/arm64/boot/dts/mediatek/mt7986a-xiaomi-redmi-router-ax6000.dts)

都使用 NMBM 坏块管理，因此只支持被启用 NMBM 的 uboot 引导程序启动。（？存疑，目前只明确看到主线 OpenWrt/ImmortalWrt 未启用 NMBM 支持故无法使用 hanwckf 的 uboot 启动的说法）

## Kernel vermagic

安装 kmod 包经常出现不兼容，原因是 openwrt 编译的时候会计算一个 kernel vermagic

在 openwrt 上查看：

```sh
opkg list-installed kernel
```

在编译目录查看编译出的内核的 vermagic

```sh
cat build_dir/target-aarch64_cortex-a53_musl/linux-mediatek_mt7986/linux-5.4.284/.vermagic
```

直接对着 `.config.set` 内核编译选项计算 vermagic

```sh
grep '=[ym]' .config.set | LC_ALL=C sort | md5sum | awk '{print $1}'
```

## Kernel module hack

由于编译的时候没加上 kmod-netlink-diag 所以只能手动 insmod

```sh
mkdir -p /lib/modules/netlink-fix/
cp /tmp/netlink_diag.ko /lib/modules/netlink-fix/
```

打开启动脚本 `/etc/rc.local` 加上这一行

```sh
insmod /lib/modules/netlink-fix/netlink_diag.ko
```

`mtd-rw.ko` 同理，用于刷写 uboot 时解锁 bl2 等分区的写入。