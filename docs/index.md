# Guild for Raspberrypi 3 to setup Openvpn + Pihole + DNS-over-HTTPS

## 1. Prerequisite
* Raspberry Pi 3 or 3+
* Internet
* Know how to use terminal and command lines

## 2. Install OS for Raspberry Pi
Read instruction at [this page](https://www.raspberrypi.org/help/noobs-setup/2/)

## 3. Openvpn
* I used PiVPN. Copy this command in terminal.

```bash
curl -L https://install.pivpn.io | bash

```
* For step-by-step, [watch this video](https://www.youtube.com/watch?v=9RSHSt4RuLk)
* After finished, reboot Pi to let tun0 show up when setup pihole.

```bash
sudo reboot
```

## 4. Pihole

### For those who already installed pihole 
* Edit /etc/pihole/setupVars.conf
```bash
sudo nano /etc/pihole/setupVars.conf
```
* Change the line `PIHOLE_INTERFACE=eth0` to `PIHOLE_INTERFACE=tun0`
* Close and save the file with Ctrl+X, enter y, enter.

### For those who hasn't install pihole yet.

#### Copy this command in terminal.
```bash
curl -sSL https://install.pi-hole.net | bash
```
#### Choose an Interface
![Interface](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/interface.PNG?raw=true "Interface")
Make sure choose tun0.

#### Choose DNS Provider
![DNS](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/dns.PNG?raw=true "DNS")
You can choose anything you want.

#### Choose Protocol
![Protocol](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/protocol.PNG?raw=true "Protocol")

#### Choose IP address
![IP address](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/IPaddress.PNG?raw=true "IP address")
Make sure IP address matches with your Pi and Gateway matches with your router.

### Web Admin Interface and log queries
You should choose `on` to easy manage logs.

## 5. Setup OpenVPN to run with Pihole

* Edit /etc/dnsmasq.conf via: `sudo nano /etc/dnsmasq.conf`.
* Modify #listen-address= to: listen-address=127.0.0.1, 192.168.x.x, 10.8.0.1.
* Replace the second IP with your Raspberry Pi local network IP and the third IP is the tun0 interface.
* Restart DNSMasq via `sudo systemctl restart dnsmasq`.
* Edit the file /etc/openvpn/server.conf via `sudo nano /etc/openvpn/server.conf`.
* Make sure to have line 10-16 like my setting below. And change `192.168.x.x` to your Pi's IP address
* If you want VPN only run query (it may help run faster if the upload speed of server is slow), comment out line 22.

```bash=
dev tun
proto udp
port 11943
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server_ow2eJpQfeONY9S4s.crt
key /etc/openvpn/easy-rsa/pki/private/server_ow2eJpQfeONY9S4s.key
dh none
topology subnet
server 10.8.0.0 255.255.255.0
# server and remote endpoints
ifconfig 10.8.0.1 10.8.0.2
# Set your primary domain name server address for clients
push "route 10.8.0.1 255.255.255.255"
push "route 10.8.0.0 255.255.255.0"
push "route 192.168.x.x 255.255.255.255" # Change to your Pi's IP adress 
push "dhcp-option DNS 192.168.x.x" # Change to your Pi's IP adress
# Prevent DNS leaks on Windows
push "block-outside-dns"
# Override the Client default gateway by using 0.0.0.0/1 and
# 128.0.0.0/1 rather than 0.0.0.0/0. This has the benefit of
# overriding but not wiping out the original default gateway.
push "redirect-gateway def1"
client-to-client
keepalive 10 120
remote-cert-tls client
tls-version-min 1.2
tls-crypt /etc/openvpn/easy-rsa/pki/ta.key
cipher AES-256-CBC
auth SHA256
compress lz4
user nobody
group nogroup
persist-key
persist-tun
crl-verify /etc/openvpn/crl.pem
status /var/log/openvpn-status.log 20
status-version 3
syslog
verb 3
#DuplicateCNs allow access control on a less-granular, per user basis.
#Remove # if you will manage access by user instead of device. 
#duplicate-cn
# Generated for use by PiVPN.io
```

* Close and save the file with Ctrl+X, enter y, enter.
* Restart OpenVPN via `sudo systemctl restart openvpn`

## 6. DNS-over-HTTPS [Optional]

* Copy these commands to terminal.
```bash
cd ~
wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.tgz
mkdir argo-tunnel
tar -xvzf cloudflared-stable-linux-arm.tgz -C ./argo-tunnel
rm cloudflared-stable-linux-arm.tgz
sudo cp ./argo-tunnel /usr/local/bin/
sudo /usr/local/bin/cloudflared proxy-dns --port 54 --upstream https://1.1.1.1/.well-known/dns-query --upstream https://1.0.0.1/.well-known/dns-query
```
* Make sure it runs like the image below.

![cloudflare](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/cloudflare.PNG?raw=true "cloudflare")
* If everything is all setup and running just fine, the last step is to make sure cloudflared is always running. To do this, we will create a systemd unit file to make sure of that.

```bash
sudo nano /etc/systemd/system/dnsproxy.service
```

```bash
[Unit]
Description=CloudFlare DNS over HTTPS Proxy
Wants=network-online.target
After=network.target network-online.target
 
[Service]
ExecStart=/usr/local/bin/cloudflared proxy-dns --port 54 --upstream https://1.1.1.1/.well-known/dns-query --upstream https://1.0.0.1/.well-known/dns-query
Restart=on-abort
 
[Install]
WantedBy=multi-user.target
```

* Close and save the file with Ctrl+X, enter y, enter.
* To ensure cloudflared runs on startup you have to enable it with the following.

```bash
sudo systemctl enable dnsproxy.service
```

* Now cloudflared will start on system boot and restart if it crashes meaning it should always be available.
