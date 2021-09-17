+++
layout = "single"
title = "Installing an alternate OpenWrt firmware on DGNWG05LM and ZHWG11LM gateways"

translationKey = "mainpage"

+++

## Table of Contents

1. [Introduction](#introduction)
2. [Gain root]({{< relref "./gain-root" >}})
3. [Making a backup](#backup)
4. [Flashing over-the-air](#flashing-over-the-air)
5. [Using OpenWrt](#using-openwrt)
6. [Working with ZigBee](#working-with-zigbee)
7. [Other software](#other-software)
8. [Resetting to defaults](#reset-to-the-defaults)
9. Soldering USB   
    * [Soldering and flashing firmware over USB]({{< relref "./usb-flashing" >}})
    * [Return to stock firmware](#return-to-stock-firmware)
10. [GPIO on the board]({{< relref "./gpio" >}})
11. [Links](#links)

## Introduction
The instruction applies only to the European version of the gateway mieu01 from 
Xiaomi, with a European plug, as well as a version of the gateway from 
Aqara ZHWG11LM with a Chinese or European plug. For xiaomi gateway2 
version with Chinese plug DGNWG02LM it will not work, it has other hardware
components installed.

This instruction assumes that you already have ssh access to the gateway.
If you have not done this, use the following instruction

[==> Gain root]({{< relref "./gain-root" >}})

## Backup
Do make a back-up copy. If you decide to return to the stock firmware,
to revert to the original firmware you need the tar.gz backup from your 
device with an archive of your root filesystem.
You cannot use a `generic backup` because every firmware contains unique 
IDs and keys. 

```shell
tar -cvpzf /tmp/lumi_stock.tar.gz -C / . --exclude='./tmp/*' --exclude='./proc/*' --exclude='./sys/*'
```

After backup is done, __download it to your local computer__.

```shell
scp root@*GATEWAY_IP*:/tmp/lumi_stock.tar.gz .
```

or using WinScp in `scp` mode (dropbear on the gateway doesn't support sftp 
mode)

If you already have a rootfs image made with dd, **make an archive anyway**.
During the boot phase of the dd image, nand flash or ubifs errors usually 
occur. Option with tar.gz does not have these drawbacks because it 
formats the flash before writing the files.

## Flashing over the air

It is the easiest and recommended way that can be used either with a serial 
console or via ssh. It doesn't require additional soldering but works only 
on the stock firmware.

Double-check you don't have any redundant archives in the `/tmp` directory.
You'll need space to download firmware binaries. The gateway must be connected
to the internet too.

Run the command:

```shell
echo -e "GET /openlumi/owrt-installer/main/install.sh HTTP/1.0\nHost: raw.githubusercontent.com\n" | openssl s_client -quiet -connect raw.githubusercontent.com:443 -servername raw.githubusercontent.com 2>/dev/null | sed '1,/^\r$/d' | bash
```

This command will stop all the processes on the gateway. It is a normal 
behavior if your ssh connection is dropped. The flashing process takes a few 
minutes. After it is done the gateway will create an open Wi-Fi network
with `OpenWrt` name.

## If, for some reason, the Over-the-Air method did not work for you, you can bring the gateway back to life by soldering usb and uart and flashing it through mfgtools
[==> Flash over USB]({{< relref "./usb-flashing" >}})

### Using OpenWrt

After flashing, the gateway will create an open Wi-Fi network with the 
name OpenWrt. To connect the gateway to your router you have to connect 
to this network and go to http://192.168.1.1/

Default credentials to the gateway are: login 'root' without a password.

Go to the section Network -> Wireless

![Go to Wireless](images/owrt_menu.png)

Press the Scan button against the first interface radio0
After a few seconds, you will see a list of networks. Find your network and
press Join Network

![Scan](images/owrt_scan.png)

In the pop-up window set the "Replace wireless configuration" checkbox.
Enter the passphrase from your Wi-Fi network below

![WiFi password](images/owrt_connect1.png)

Confirm the settings on the next window, press the Save button.

![WiFi password-2](images/owrt_connect2.png)

To apply the changes correctly, you should disable Access Poing by pressing 
"Disable" button against connection for the second interface.

![Disable AP](images/owrt_disable_ap.png)

The gateway will disconnect you from AP and will apply the changes.
After the firmware, the mac address of the gateway changes, because the ip address is also most likely
will change. Check it in the router or in the gateway itself.

The gateway is pre-installed:
- OpenWrt LuCi GUI on port 80 http
- command utility for flashing zigbee module jn5169
- Web plugin for LuCi to flash a firmware

Do not enable Wi-Fi AP + Station modes on the gateway at the same time.
The driver that is used in the system cannot work in two modes
at the same time.
If you changed the LuCi settings and after that the gateway stopped connecting to the network,
press the button on the gateway for 10 seconds. It will blink yellow 3 times
and with start the initial network configuration mode with creating Wi-Fi 
Access Point.

### Working with ZigBee

Zigbee chip can work only with a single system, therefore you have to choose
a program you'd like to use. But at the same time, you can use zigbee2mqtt to 
work with zigbee and domoticz for other automations.

1. [Installing Zigbee2mqtt]({{< relref "./zigbee2mqtt" >}})
2. [Installing Home Assistant with ZHA component](https://github.com/openlumi/homeassistant_on_openwrt)   
3. [Installing Zesp32]({{< relref "./zesp32" >}})
4. [Installing Domoticz and configuring Zigate plugin]({{< relref "./domoticz-zigate" >}})

### Other software

1. [https://github.com/openlumi/lumimqtt/](https://github.com/openlumi/lumimqtt/) - a service that allow managing the gateway devices over the MQTT
2. [https://flows.nodered.org/node/node-red-contrib-xiaomi-gateway](https://flows.nodered.org/node/node-red-contrib-xiaomi-gateway) - a package for node red

### Reset to the defaults

To erase data on the OpenWrt firmware and go to the initial set up 
(like you just flashed the gateway), you must hold the gateway button for
20 seconds. The gateway will blink red 3 times and will reset to the initial
set up with creating Wi-Fi Access Point.
Be careful with resetting, all programs and settings will be erased.
Use it in case of emergency, when resetting Wi-Fi credentials not help.


### Return to stock firmware

To return to the stock firmware you have to flash original kernel, dtb
and rootfs from your backup. Kernel and DTB are the same for all gateways
and to keep the Mi Home working, you'll need your tar.gz backup.

[mfgtools to return to the stock firmware](files/mfgtools-lumi-stock.zip)

Put your backup with the name`lumi_stock.tar.gz` to directory
`Profiles/Linux/OS Firmware/files` overwriting the empty file
`lumi_stock.tar.gz`

Then again put the gateway into the boot mode via usb and via mfgtools
flash the original firmware.

To flash zigbee firmware back, you should log in to the gateway with stock
firmware and run the command
```
touch /home/root/need_update_coordinator.tag 
```
Then reboot. The gateway will automatically restore Zigbee firmware when 
started.

## Links

1. An article that details the changes and technical modifications:
[Xiaomi Gateway (eu version - Lumi.gateway.mieu01) Hacked] (https://habr.com/ru/post/494296/)
2. Collection of information on hardware and software modding of Xiaomi Gateway [https://github.com/T-REX-XP/XiaomiGatewayHack](https://github.com/T-REX-XP/XiaomiGatewayHack)
2. Telegram channel with discussion of modifications [https://t.me/xiaomi_gw_hack ](https://t.me/xiaomi_gw_hack)
