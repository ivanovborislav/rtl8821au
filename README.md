-----

## rtl8821au (8821au.ko)

## Realtek RTL8821AU Wireless Lan Driver for Linux

- v5.12.5.2 (20210708)
- Based on EDIMAX EW-7811UTC Linux Driver (Version : 1.0.3.4) 2021-09-16
- Support Kernel: 4.15 - 5.11 (Realtek)
- Support up to Kernel 5.15

##	Specification

- Supported interface modes:
  * IBSS
  * managed
  * AP
  * monitor
  * P2P-client
  * P2P-GO
- Packet injection
- TX power control
- LED control
- Power saving control
- Driver debug log level control
- VHT control
- Security:
  * WEP 64/128-bit, WPA, WPA2, WPA3, and 802.1x

## HOW TO

### Install

Download source:

```
git clone https://github.com/ivanovborislav/rtl8821au.git
cd rtl8821au
```

Install missing packages:

```
sudo apt-get install bc build-essential
```

Install linux headers:

```
sudo apt-get install linux-headers-$(uname -r)
```

or

```
apt-cache search linux-headers
sudo apt-get install linux-headers-5.14.0-kali4-amd64 (for example)
apt-cache search linux-image
sudo apt-get install linux-image-5.14.0-kali4-amd64 (for example)
```

Compile:

```
make
sudo make install
```

### Monitor mode

```
sudo airmon-ng check kill
sudo ip link set <interface> down
sudo iw dev <interface> set type monitor
sudo ip link set <interface> up
```

### Managed mode

```
sudo ip link set <interface> down
sudo iw dev <interface> set type managed
sudo ip link set <interface> up
sudo systemctl restart NetworkManager (sudo service network-manager restart)
```

### TX power control

Note: Set TX power before start SoftAP mode. `...set txpower fixed 3000 = txpower 30.00 dBm`.

```
sudo iw dev <interface> set txpower fixed 3000
```

### Driver options

#### Change driver options during inserting driver module

Remove (unload) a module from the Linux kernel.
```
sudo rmmod /lib/modules/$(uname -r)/kernel/drivers/net/wireless/8821au.ko
```

Insert (load) a module into the Linux kernel.
```
sudo insmod /lib/modules/$(uname -r)/kernel/drivers/net/wireless/8821au.ko rtw_ips_mode=1 rtw_drv_log_level=4 rtw_power_mgnt=2 rtw_led_ctrl=1
```

#### Change driver options loading from file

Create a file `8821au.conf` containing `options 8821au rtw_ips_mode=1 rtw_drv_log_level=4 rtw_power_mgnt=2 rtw_led_ctrl=1`.
Copy a file to `/etc/modprobe.d/` directory.

```
sudo cp -f 8821au.conf /etc/modprobe.d
```

Power saving control.

IPS (Inactive Power Saving) Function, rtw_ips_mode=
```
0:Disable IPS
1:Enable IPS (default)
```

LPS (Leisure Power Saving) Function, rtw_power_mgnt=
```
0:Disable LPS
1:Enable LPS
2:Enable LPS with clock gating (default)
```

Driver debug log level control, rtw_drv_log_level=
```
0:_DRV_NONE_
1:_DRV_ALWAYS_
2:_DRV_ERR_
3:_DRV_WARNING_
4:_DRV_INFO_ (default)
5:_DRV_DEBUG_
6:_DRV_MAX_
```

Driver LED control, rtw_led_ctrl=
```
0:led off
1:led blink (default)
2:led on
```

Driver VHT control, rtw_vht_enable=
```
0:disable
1:enable (default)
2:force auto enable
```

### Connecting with wpa_supplicant

Example wpa_supplicant.conf with WPA3-Personal (WPA3-SAE).

```
update_config=1
ctrl_interface=/var/run/wpa_supplicant
country=EN
p2p_no_group_iface=1
sae_groups=19 20 21

network={
	ssid="WPA3"
	proto=RSN
	key_mgmt=SAE
	pairwise=CCMP
	group=CCMP
	ieee80211w=2
	psk="1234567890"
}
```

Now start...
```
sudo systemctl stop NetworkManager
sudo killall wpa_supplicant
sudo wpa_supplicant -B -i <interface> -c wpa_supplicant.conf
sudo dhclient <interface>
```

### Start SoftAP mode

Example hostapd.conf with WPA3-Personal (WPA3-SAE) 2.4GHz.

```
driver=nl80211
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0
ssid=WPA3
country_code=EN
hw_mode=g
channel=6
beacon_int=100
dtim_period=1
max_num_sta=16
rts_threshold=2347
fragm_threshold=2346
ignore_broadcast_ssid=0
wmm_enabled=1
ieee80211n=1
ht_capab=[HT40-][SHORT-GI-20][SHORT-GI-40][RX-STBC1][MAX-AMSDU-7935][DSSS_CCK-40]

auth_algs=1
wpa=2
wpa_passphrase=1234567890
wpa_key_mgmt=SAE
wpa_pairwise=CCMP
rsn_pairwise=CCMP
ieee80211w=2
sae_groups=19 20 21
sae_require_mfp=1
````

Example hostapd.conf with WPA3-Personal (WPA3-SAE) 5GHz.

CAUTION: Allow width: 80 MHz, `insmod 8821au.ko rtw_vht_enable=2`.

```
driver=nl80211
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0
ssid=WPA3_5GHz
country_code=EN
hw_mode=a
channel=36
beacon_int=100
dtim_period=1
max_num_sta=16
rts_threshold=2347
fragm_threshold=2346
ignore_broadcast_ssid=0
wmm_enabled=1
ieee80211n=1
ht_capab=[HT40+][SHORT-GI-20][SHORT-GI-40][MAX-AMSDU-7935][DSSS_CCK-40]
ieee80211ac=1
vht_capab=[MAX-MPDU-11454][SHORT-GI-80][RX-STBC1][SU-BEAMFORMEE][HTC-VHT][MAX-A-MPDU-LEN-EXP7]
vht_oper_chwidth=1
vht_oper_centr_freq_seg0_idx=42

auth_algs=1
wpa=2
wpa_passphrase=1234567890
wpa_key_mgmt=SAE WPA-PSK
wpa_pairwise=CCMP
rsn_pairwise=CCMP
ieee80211w=2
sae_groups=19 20 21
sae_require_mfp=1
```

Now start...
```
sudo killall hostapd
sudo hostapd -i <interface> hostapd.conf
```

## Test devices

- TP-Link Archer T2U Plus V1

- Linksys WRT1200AC V2
  * OpenWrt 21.02.0 r16279-5cc0535800 / LuCI openwrt-21.02 branch git-21.231.26241-422c175

```
config wifi-iface 'default_radio1'
	option device 'radio1'
	option network 'lan'
	option mode 'ap'
	option macaddr '30:23:03:XX:XX:XX'
	option ssid 'WPA3'
	option encryption 'sae'
	option key '1234567890'
	option ieee80211w '2'
```

-----
