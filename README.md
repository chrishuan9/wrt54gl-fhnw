# WRT54GL as an 802.1x client (aka wrt54gl@eduroam)

Simple HowTo describing the process of running a Linksys WRT54GL as a lan/wifi client with 802.1X authentication. It can be used for accessing the eduroam (educational network initiative) network.

OpenWRT by default does not support 802.1x, but fortunately a
wpa_supplicant package is available already precompiled for this platform.

wpa_supplicant is an EAP supplicant with 802.1x authentication + wpa/wpa2 support.

(In case you’re running OpenWRT on a different chipset/router make sure
that the wpa_supplicant is available precomipled (compiling it is
difficult)

## Installing OpenWRT

The last OpenWRT version that is fully supported on the WRT54GL is 10.03.1.
As the WRT54GL has only 4Mb flash, any image sent to the device must be 3866624 bytes or smaller.

Use branch brcm-2.4: For versions of the OpenWrt "brcm47xx" target prior to
"Attitude Adjustment" 12.09-final, you may wish to use Broadcom's proprietary wl driver due to
longstanding issues with the b43 driver in Linux kernel versions 2.6 and newer (https://dev.openwrt.org/ticket/7552).

* Download: http://downloads.openwrt.org/backfire/10.03/brcm-2.4/openwrt-wrt54g-squashfs.bin

* Used the built-in Linksys firmware installer, pointed it at the .bin file and update to openwrt.

## First Login

* Connect to the router using telnet 192.168.1.1 to setup a root password:

BusyBox v1.15.3 (2011-11-24 02:12:13 CET) built-in shell (ash)
Enter 'help' for a list of built-in commands.

```
  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 Backfire (10.03.1, r29592) ------------------------
  * 1/3 shot Kahlua    In a shot glass, layer Kahlua
  * 1/3 shot Bailey's  on the bottom, then Bailey's,
  * 1/3 shot Vodka     then Vodka.
 ---------------------------------------------------
```

* type ``` passwd ``` into the prompt. You will be prompted to set a new password for the user root:

```
root@openwrt:~$ passwd
Changing password for root
New password:
Retype password:
Password for root changed by root
root@openwrt:~$
```
* after you set a password the telnet daemon will be disabled, type ```exit``` into the prompt
* without reboot, SSH is now available; so is HTTPS if the WebUI (LuCI) is installed with it's TLS-modules
* login again with ssh root@192.168.1.1

## Install WPA Supplicant package:

* opkg install http://downloads.openwrt.org/backfire/10.03/brcm-2.4/packages/wpa-supplicant_20100309-1_brcm-2.4.ipk

 or

* opkg update && opkg install wpa-supplicant

### Create or copy (via scp) WPA supplicant configuration file

/etc/config/wpa.conf

```
ctrl_interface=/var/run/wpa_supplicant
ap_scan=0
network={
key_mgmt=IEEE8021X
eap=PEAP
identity="edu\yourusername"
password="yourpassword"
#phase1="peaplabel=0"
phase2="auth=MSCHAPV2"
}
```

Try to authenticate on the networking by issuing the following command:

``` wpa_supplicant -i eth0.1 -D roboswitch -c /etc/config/wpa.conf ```

roboswitch is the family name of the driver for the broadcom switch set required to authenticate.

## Automatic logon on boot-up

Place the following start-up script into /etc/init.d/

```
#!/bin/sh /etc/rc.common
# FHNW Logon script
# Last Modified 2014/03/07
# author chris.yereztian@students.fhnw.ch

START=99

start(){

# Set time - otherwise the default WRT's time makes problems
# with a certificate validation in wpa_supplicant
#date "`cat /etc/dateToSet`"
#date "`cat /etc/dateToSet.backup`"

echo "Signing into FHNW-Network"
wpa_supplicant -i eth0.1 -D roboswitch -c /etc/config/wpa.conf &
sleep 15
udhcpc -i eth0.1 &

# sync time with the NTP server and store the local
# time into a file for reboot use
/etc/updateDate &
}
```
802.1x authentication is time sensitive. It is recommended to sync the time using an ntp server and store it locally so the first authentication on boot-up won't fail. This is necessary since the WRT54GL does not have an internal clock and will lose track upon a reboot.

* Invoke the enable command to run the script on boot-up

```root@OpenWrt:/# /etc/init.d/signinfhnw enable```
