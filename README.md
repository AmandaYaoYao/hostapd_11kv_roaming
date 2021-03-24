# hostapd_11kv_roaming
This project is to test the 11k/v roaming protocol with hostapd.

# Platform

* Ubuntu 16.04

## Requirements

```
sudo apt-get install pkg-config libnl-3-dev libssl-dev libnl-genl-3-dev

wget https://w1.fi/releases/hostapd-2.9.tar.gz
tar xzvfhostapd-2.9.tar.gz
cd hostapd-2.9/hostapd
cp defconfig .config
make
sudo make install

hostapd -v
-> hostapd v2.9
hostapd_cli -v 
-> hostapd_cli v2.9
```

## Setup the environment

* create a hostapd config file. Here I named it to `11kv.config`
```
touch 11kv.config
```
* copy the content below to the config file (Here is WPA2 entercription, others can refer to the config files in the resposity). The wlan0 is the interface in you PC.

```
ctrl_interface=/var/run/hostapd
interface=wlan0
driver=nl80211
hw_mode=g
country_code=US
wmm_enabled=1
macaddr_acl=0
ssid=roam
wpa_passphrase=1234567890
channel=6
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
rrm_neighbor_report=1
rrm_beacon_report=1
bss_transition=1
```
* setup the dhcp, choose the correct port and ip range. Here I set the port to 4242 and the IP range is 192.168.1.1,192.168.1.20.

```
 dnsmasq --no-daemon --no-resolv --no-poll --dhcp-range=192.168.1.1,192.168.1.20,1h -p 4242
```
* set the wireless interface ip address.

```
sudo ifconfig wlan0 192.168.1.1 up
```

* Set up the AP with hostapd

```
sudo hostapd 11kv.config
```

* Connect to the setup AP with your target device
* There should be another AP has the same ssid and password as the AP you setup with hostapd
* Send BTM or RM related frame with hostapd_cli


```
sudo hostapd_cli -i wlan0

# RM
req_beacon <station mac> 51000000640001ffffffffffff

# BTM
bss_tm_req <station mac> disassoc_imminent=1 disassoc_timer=20
bss_tm_req <station mac> pref=1 neighbor=<target AP bssid>,0x0000,81,1,7,0301ff
```

