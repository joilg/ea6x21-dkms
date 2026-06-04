# ea6x21-dkms
wifi driver for EA6521 and EA6621 WIFI driver

# Seekwave Wifi6  (ea6x21q) Linux Driver 

The driver was tested on a X88PRO13 TV box  under Armbian  with RK3525 CPU and EA6521 Wifi Chip. 
It should be compatible with other WiFi SDIO adapters with the same chip of EA6x21 inside.  

Note that the kernel should has configuration as below.  This is default config in Armbian or Debian
```
[*] Networking support --->
      <*>   Wireless --->
              <*>   cfg80211 - wireless configuration API
              <*>   Generic IEEE 802.11 Networking Stack (mac80211)
      <*>   RF switch subsystem support --->
    Device Drivers --->
      [*] Network device support --->
            [*]   Wireless LAN --->
      [*] Staging drivers --->
            <M>   Support for rtllib wireless devices
            <M>     Support for rtllib CCMP crypto
            <M>     Support for rtllib TKIP crypto
            <M>     Support for rtllib WEP crypto
```

## Device Tree:

### add this to your device Tree source file (dts)
tis example is for the X88PRO13 TV box. __ modyfy this to our Board specific Hardware__
```
seekwcn_boot>;
	compatible = "seekwave,sv6160";
	dma_type = <0x01>;
	skw_iram_path = "/lib/firmware/SWT6621_IRAM_SDIO.bin";
	skw_dram_path = "/lib/firmware/SWT6621_DRAM_SDIO.bin";
	bt_antenna = <0>;   /* no BT_antenna setting */
	// seekwave_nv_name = "SEEKWAVE_NV_SWT6652.bin";
	gpio_host_wake = <50>;                      // __Insert here your Board specific GPIO __
	gpio_chip_wake = <49>;                       // __Insert here your Board specific GPIO __
	gpio_chip_en =	  <38>;                       // __Insert here your Board specific GPIO __
	pinctrl-names = "default";
	status = "okay";
};
```
### or apply this device tree overlay:   


```
/dts-v1/;
/plugin/;
/ {
	compatible = "rockchip,rk3528";

    fragment@0 {
        target = <&seekwcn_boot>;
        __overlay__ {
			compatible = "seekwave,sv6160";
			dma_type = <0x01>;
			skw_iram_path = "/lib/firmware/SWT6621_IRAM_SDIO.bin";
			skw_dram_path = "/lib/firmware/SWT6621_DRAM_SDIO.bin";
			bt_antenna = <0>;   /* no BT_antenna setting */
			// seekwave_nv_name = "SEEKWAVE_NV_SWT6652.bin";
			gpio_host_wake = <50>;
			gpio_chip_wake = <49>; 
			gpio_chip_en =	  <38>; 
			pinctrl-names = "default";
			status = "okay";
		};
	};
};

```
Apply overlay with 
```
  sudo armbian-add-overlay rk35xx_openvfd.dts 
  sudo reboot  
```
## Install driver:
```
git clone https://github.com/joilg/dkms-ea6x21.git

sudo cp -r dkms-ea6x21q/ea6x21p-1.0 /usr/src

sudo dkms add -m ea6621q -v 1.0
sudo dkms build -m ea6621q -v 1.0
sudo dkms install -m ea6621q -v 1.0 
```
## load driver 

modprobe skw_sdio
modprobe skw_bootcoms
modprobe skw
modprobe skwbt

### to load on startup
cat <<EOF > /etc/modules-load.d/skw.conf
hidp
rfcomm
bnep
skw_sdio
skw_bootcoms
skw
skwbt
EOF


## troubleshooting

### Test wifi chip
```## sudo journalctl -b | grep SDIO
       kernel: mmc2: new ultra high speed SDR104 SDIO card at address 8800
```

```
## sudo ip link 
...
...
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff permaddr xx:xx:xx:xx:xx:xx
    altname xxxxxxxxxxxx
...
```

## uninstall: 
```
dkms remove -m ea6621q -v 1.0
```


## Contributing

Feel free to dive in! [Open an issue](https://github.com/joilg/ea6x21/issues/new) or submit PRs.


