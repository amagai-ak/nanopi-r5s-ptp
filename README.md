# nanopi-r5s-ptp

Setup nanopi R5S as a NTP/PTP server.
The OS I use here is friendlycore-focal-arm64.

## 1. Prepare hardwares

![cable](img/cable.png)

### 1.1 Buy GPS receiver

I use GPS device with serial connection, not USB connection. And the GPS device must have a 1-PPS output. The GPS receiver must be powered by 3.3v and the signal level must also be 3.3v. 

### 1.2 Buy GPIO cable

12-pin 0.5mm FPC cable like this.

https://www.amazon.co.jp/dp/B09V2K1S2C

FPC connector to 2.54 mm pitch conversion PCB for soldering.

https://www.amazon.co.jp/dp/B07H2CN4TZ/

### 1.3 Assemble them

| Pin# | GPIO | UART | GPS |
| ---- | ---- | ---- | --- |
| 1 | 3V3 | | 3V3   |
| 3 | GPIO3_C3 | UART5_RX | TXD |
| 4 | GND |  | GND |
| 5 | GPIO3_C2 | UART5_TX | RXD |
|12 | GPIO3_C5 | | 1 PPS |

See below for more information of GPIO.

https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R5S

## 2. Build initial image

https://github.com/friendlyarm/sd-fuse_rk3568

Clone above repository and follow it's build instructions.

Succeeded to build SD-Card image?

Okay, let's modify it for PTP.

## 3. Modify scripts for PTP support

### 3.1 Overwrite kernel configuration

Copy config/nanopi5_linux_defconfig of this repo into sd-fuse_rk3568/out/kernel-rk3568/arch/arm64/configs

### 3.2 Modify device tree

To enable UART5 and pps-gpio, modify the device tree file 'sd-fuse_rk3568/out/kernel-rk3568/arch/arm64/boot/dts/rockchip/rk3568-nanopi5-rev01.dts'.

Overwrite that file with "dts/rk3568-nanopi5-rev01.dts" of this repository.

### 3.3 Modify Makefile for R8125

Open 'sd-fuse_rk3568/out/r8125/Makefile' with editor and change PTP related lines.
```
ENABLE_PTP_SUPPORT = y
ENABLE_PTP_MASTER_MODE = y
```

## 4. Build & write SD-Card image and boot with it

```
sudo ./build-kernel.sh friendlycore-focal-arm64
./mk-sd-image.sh friendlycore-focal-arm64
```

Boot nanopi R5S with that SD-Card. Did it boot successfully?

The following operations are performed on nanopi-R5S with the newly created SD card inserted.

## 5. Check ethernet functions

Install ethtool.

```
sudo apt-get install ethtool
```

Check that each LAN interface supports PTP hardware time stamping.


```
$ ethtool -T eth0
Time stamping parameters for eth0:
Capabilities:
        hardware-transmit     (SOF_TIMESTAMPING_TX_HARDWARE)
        software-transmit     (SOF_TIMESTAMPING_TX_SOFTWARE)
        hardware-receive      (SOF_TIMESTAMPING_RX_HARDWARE)
        software-receive      (SOF_TIMESTAMPING_RX_SOFTWARE)
        software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
        hardware-raw-clock    (SOF_TIMESTAMPING_RAW_HARDWARE)
PTP Hardware Clock: 0
Hardware Transmit Timestamp Modes:
        off                   (HWTSTAMP_TX_OFF)
        on                    (HWTSTAMP_TX_ON)
Hardware Receive Filter Modes:
        none                  (HWTSTAMP_FILTER_NONE)
        all                   (HWTSTAMP_FILTER_ALL)
        ptpv1-l4-event        (HWTSTAMP_FILTER_PTP_V1_L4_EVENT)
        ptpv1-l4-sync         (HWTSTAMP_FILTER_PTP_V1_L4_SYNC)
        ptpv1-l4-delay-req    (HWTSTAMP_FILTER_PTP_V1_L4_DELAY_REQ)
        ptpv2-l4-event        (HWTSTAMP_FILTER_PTP_V2_L4_EVENT)
        ptpv2-l4-sync         (HWTSTAMP_FILTER_PTP_V2_L4_SYNC)
        ptpv2-l4-delay-req    (HWTSTAMP_FILTER_PTP_V2_L4_DELAY_REQ)
        ptpv2-event           (HWTSTAMP_FILTER_PTP_V2_EVENT)
        ptpv2-sync            (HWTSTAMP_FILTER_PTP_V2_SYNC)
        ptpv2-delay-req       (HWTSTAMP_FILTER_PTP_V2_DELAY_REQ)

```

```
$ ethtool -T eth1
Time stamping parameters for eth1:
Capabilities:
        hardware-transmit     (SOF_TIMESTAMPING_TX_HARDWARE)
        software-transmit     (SOF_TIMESTAMPING_TX_SOFTWARE)
        hardware-receive      (SOF_TIMESTAMPING_RX_HARDWARE)
        software-receive      (SOF_TIMESTAMPING_RX_SOFTWARE)
        software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
        hardware-raw-clock    (SOF_TIMESTAMPING_RAW_HARDWARE)
PTP Hardware Clock: 1
Hardware Transmit Timestamp Modes:
        off                   (HWTSTAMP_TX_OFF)
        on                    (HWTSTAMP_TX_ON)
Hardware Receive Filter Modes:
        none                  (HWTSTAMP_FILTER_NONE)
        ptpv2-l4-event        (HWTSTAMP_FILTER_PTP_V2_L4_EVENT)
        ptpv2-l4-sync         (HWTSTAMP_FILTER_PTP_V2_L4_SYNC)
        ptpv2-l4-delay-req    (HWTSTAMP_FILTER_PTP_V2_L4_DELAY_REQ)
        ptpv2-event           (HWTSTAMP_FILTER_PTP_V2_EVENT)
        ptpv2-sync            (HWTSTAMP_FILTER_PTP_V2_SYNC)
        ptpv2-delay-req       (HWTSTAMP_FILTER_PTP_V2_DELAY_REQ)
```

```
$ ethtool -T eth2
Time stamping parameters for eth2:
Capabilities:
        hardware-transmit     (SOF_TIMESTAMPING_TX_HARDWARE)
        software-transmit     (SOF_TIMESTAMPING_TX_SOFTWARE)
        hardware-receive      (SOF_TIMESTAMPING_RX_HARDWARE)
        software-receive      (SOF_TIMESTAMPING_RX_SOFTWARE)
        software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
        hardware-raw-clock    (SOF_TIMESTAMPING_RAW_HARDWARE)
PTP Hardware Clock: 2
Hardware Transmit Timestamp Modes:
        off                   (HWTSTAMP_TX_OFF)
        on                    (HWTSTAMP_TX_ON)
Hardware Receive Filter Modes:
        none                  (HWTSTAMP_FILTER_NONE)
        ptpv2-l4-event        (HWTSTAMP_FILTER_PTP_V2_L4_EVENT)
        ptpv2-l4-sync         (HWTSTAMP_FILTER_PTP_V2_L4_SYNC)
        ptpv2-l4-delay-req    (HWTSTAMP_FILTER_PTP_V2_L4_DELAY_REQ)
        ptpv2-event           (HWTSTAMP_FILTER_PTP_V2_EVENT)
        ptpv2-sync            (HWTSTAMP_FILTER_PTP_V2_SYNC)
        ptpv2-delay-req       (HWTSTAMP_FILTER_PTP_V2_DELAY_REQ)
```

## 6. Check GPS connection

Make sure that the ttyS5 and pps devices are present.

```
$ ls /dev/ttyS*
/dev/ttyS5
$ ls /dev/pps*
/dev/pps0  /dev/pps1  /dev/pps2
```

Install screen command as a comm terminal.

```
sudo apt-get install screen
```

If the baud rate of your GPS device is configured to 115200, then type below to check the serial port.

```
sudo screen /dev/ttyS5 115200
```

Do you see NMEA messages on your screen? When the GPS fixes (it takes more than 10 minutes), you will find latitude and longitude in the message. Then 1-PPS signal may be generated.
Press Ctrl-A then 'k' to exit.

Now install ppstool to check 1-PPS signal.

```
sudo apt-get install pps-tools
```

```
sudo ppstest /dev/pps0
```

If it says, "Timeout", the 1-PPS signal is not received. Check that the 1PPS signal is output with an oscilloscope.

