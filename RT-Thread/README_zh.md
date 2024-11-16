# RT-Thread on Milk-V Duo S 测试报告

## 测试环境

### 操作系统信息

- 源码链接：https://github.com/RT-Thread/rt-thread
- 参考安装文档：https://github.com/RT-Thread/rt-thread/tree/master/bsp/cvitek
- 工具链：https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1705395512373/Xuantie-900-gcc-elf-newlib-x86_64-V2.8.1-20240115.tar.gz

### 硬件信息

- Milk-V Duo S
- USB-A to C 或 USB C to C 线缆一条
- microSD 卡一张
- USB to UART 调试器一个（如：CH340, CH341, FT2232 等）

## 构建步骤

该构建流程在Ubuntu22.04上成功构建，并且在其他主流的Linux发行版可以采用同样的方法构建

### 拉取源码并编译固件

获取工具链并配置：
```bash
wget https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1705395512373/Xuantie-900-gcc-elf-newlib-x86_64-V2.8.1-20240115.tar.gz

tar -xzvf Xuantie-900-gcc-elf-newlib-x86_64-V2.8.1-20240115.tar.gz
```

自行更改以下路径：
```bash
export RTT_CC_PREFIX=riscv64-unknown-elf-
export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.8.1/bin
```

获取依赖：
```bash
sudo apt install -y scons libncurses5-dev device-tree-compiler
# 在 Arch Linux 上为：sudo pacman -S scons dtc ncurses
```

```shell
git clone --depth=1 https://github.com/RT-Thread/rt-thread
cd rt-thread/bsp/cvitek/cv18xx_risc-v
# 生成配置文件
scons --menuconfig
source ~/.env/env.sh
pkgs --update
scons -j$(nproc) --verbose
cd ../
./combine-fip.sh $(pwd)/cv18xx_risc-v Image
```

menuconfig 中的 Board Type 请选择 `milkv-duos`。

执行结束后，会在 `output` 目录下生成 boot.sd 和 fip.bin 两个文件。

### 准备 microSD 卡

清空 microSD 卡，并创建一个 FAT32 分区：
```shell
wipefs -af /path/to/your-card
mkfs.fat /path/to/your-card
```

将构建出的 boot.sd 和 fip.bin 复制进 microSD 卡。至此，存储卡已经可用来在 Duo S 上启动 RT-Thread。

### 登录系统

通过串口登录系统。

## 预期结果

系统正常启动，能够通过串口登录。

## 实际结果

系统正常启动，成功通过串口登录。

### 启动信息

```log

C�Өu��r�W��%����T�ҵ����                 ��/�e����Ő��r��B����    ��0���a2���a���0�BQ���
FSBL Jb2829:g7970f2b7:2024-11-05T16:32:02+08:00
st_on_reason=d0000
st_off_reason=0
P2S/0x1000/0xc00a200.
SD/0x9200/0x1000/0x1000/0.P2E.
DPS/0xa200/0x2000.
SD/0xa200/0x2000/0x2000/0.DPE.
cv181x DDR init.
ddr_param[0]=0x78075562.
pkg_type=1
D2_4_1
DDR3-4G-BGA
Data rate=1866.
DDR BIST PASS
PLLS/OD.
C2S/0xc200/0x9fe00000/0x11200.
SD/0xc200/0x11200/0x11200/0.RSC.
C2E.
MS/0x1d400/0x80000000/0x1b000.
SD/0x1d400/0x1b000/0x1b000/0.ME.
L2/0x38400.
SD/0x38400/0x200/0x200/0.L2/0x414d3342/0xcafeee6c/0x80200000/0x37400/0x37400
COMP/1.
SD/0x38400/0x37400/0x37400/0.DCP/0x80200020/0x1000000/0x81900020/0x37400/1.
DCP/0x73c7a/0.
Loader_2nd loaded.
Switch RTC mode to xtal32k
Jump to monitor at 0x80000000.
OPENSBI: next_addr=0x80200020 arg1=0x80080000
OpenSBI v0.9
   __                    ___ __ ___
  / __ \                  / ____|  _ \_   _|
 | |  | |_    _ _  | (_ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name             : Milk-V DuoS
Platform Features         : mfdeleg
Platform HART Count       : 1
Platform IPI Device       : clint
Platform Timer Device     : clint
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform SysReset Device  : ---
Firmware Base             : 0x80000000
Firmware Size             : 132 KB
Runtime SBI Version       : 0.3

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000074000000-0x000000007400ffff (I)
Domain0 Region01          : 0x0000000080000000-0x000000008003ffff ()
Domain0 Region02          : 0x0000000000000000-0xffffffffffffffff (R,W,X)
Domain0 Next Address      : 0x0000000080200020
Domain0 Next Arg1         : 0x0000000080080000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART ISA             : rv64imafdcvsux
Boot HART Features        : scounteren,mcounteren,time
Boot HART PMP Count       : 16
Boot HART PMP Granularity : 4096
Boot HART PMP Address Bits: 38
Boot HART MHPM Count      : 8
Boot HART MHPM Count      : 8
Boot HART MIDELEG         : 0x0000000000000222
Boot HART MEDELEG         : 0x000000000000b109


U-Boot 2021.10 (Nov 05 2024 - 16:31:57 +0800) cvitek_cv181x

DRAM:  510 MiB
gd->relocaddr=0x951c8000. offset=0x14fc8000
set_rtc_register_for_power
MMC:   cv-sd@4310000: 0, wifi-sd@4320000: 1
Loading Environment from nowhere... OK
In:    serial
Out:   serial
Err:   serial
Net:   
Warning: ethernet using random MAC address - b6:0a:c2:89:7b:22
eth0: ethernet@4070000
Hit any key to stop autoboot:  0 
Boot from SD ...
mmc1 : finished tuning, code:54
switch to partitions #0, OK
mmc0 is current device
166188 bytes read in 3 ms (52.8 MiB/s)
## Loading kernel from FIT Image at 81800000 ...
   Using 'config-cv1813h_milkv_duos_sd' configuration
   Trying 'kernel-1' kernel subimage
     Description:  cvitek kernel
     Type:         Kernel Image
     Compression:  lzma compressed
     Data Start:   0x818000d8
     Data Size:    143658 Bytes = 140.3 KiB
     Architecture: RISC-V
     OS:           Linux
     Load Address: 0x80200000
     Entry Point:  0x80200000
     Hash algo:    crc32
     Hash value:   28c2d29c
   Verifying Hash Integrity ... crc32+ OK
## Loading fdt from FIT Image at 81800000 ...
   Using 'config-cv1813h_milkv_duos_sd' configuration
   Trying 'fdt-cv1813h_milkv_duos_sd' fdt subimage
     Description:  cvitek device tree - cv1813h_milkv_duos_sd
     Type:         Flat Device Tree
     Compression:  uncompressed
     Data Start:   0x81823318
     Data Size:    20656 Bytes = 20.2 KiB
     Architecture: RISC-V
     Hash algo:    sha256

Geku, [11/6/24 4:42 PM]
Hash value:   3b359b93e003c92d0dae248650133d12d98901ee5b8fe67eabf093fc05f2fc2a
   Verifying Hash Integrity ... sha256+ OK
   Booting using the fdt blob at 0x81823318
   Uncompressing Kernel Image
   Decompressing 398440 bytes used 44ms
   Loading Device Tree to 0000000094878000, end 00000000948800af ... OK

Starting kernel ...

[I/drv.pinmux] Pin Name = "UART0_RX", Func Type = 281, selected Func [0]

[I/drv.pinmux] Pin Name = "UART0_TX", Func Type = 282, selected Func [0]

heap: [0x80292458 - 0x81200000]

 \ | /
- RT -     Thread Operating System
 / | \     5.2.0 build Nov  5 2024 15:37:10
 2006 - 2024 Copyright by RT-Thread team
lwIP-2.1.2 initialized!
[I/sal.skt] Socket Abstraction Layer initialize success.
Hello RISC-V!
msh />
```
开始有乱码属于正常现象

## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

成功
