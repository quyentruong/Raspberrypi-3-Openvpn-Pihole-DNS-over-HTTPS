# Guide for Oracle Cloud to setup Openvpn + Pihole

## 1. Prerequisite
* Oracle Cloud free tier: https://www.oracle.com/cloud/free
* Internet
* Know how to use terminal and command lines

## 2. Install Ubuntu in Oracle cloud
* Setup a VM instance
  * Switch the OS from Oracle Linux to Canonical Ubuntu (20+)
  * Click on `Save Private Key` or Upload your own public key (this key can be generated via PuTTY)
  * Note down Public IP and Private IP

* In the Instance information, click on `subnet-xxx` -> `Default Security List xxx` -> `Add Ingrss Rules`
  * Source CIDR: 0.0.0.0/0
  * IP Protocol: UDP
  * Destination Port: 1194  

## 3. Install Openvpn
* I used PiVPN. Need root access. Copy these commands in terminal.

```bash
sudo su - root
curl -L https://install.pivpn.io | bash
```

* For step-by-step, [watch this video](https://www.youtube.com/watch?v=9RSHSt4RuLk). I used default setting like the video. Only part about ip, I chose 10.8.0.1. And about port, I chose default port 1194.
* After finished, reboot Pi to let tun0 show up when setup pihole.

```bash
sudo reboot
```

## 4. Install Pihole

Copy these commands in terminal.

```bash
sudo su - root
curl -sSL https://install.pi-hole.net | bash
```
Make sure choose tun0. Everything else should be similar to the last post

## 5. Setup OpenVPN to run with Pihole

Copy these commands in terminal.

```bash
sudo su - root
curl -L https://install.pivpn.io | bash
```
* Then choose Reconfigure. Just go through the setup again. There will be a message that detects pihole. It will help configure openvpn to work with pihole. Then reboot.

* If you want VPN only run query (it may help run faster if the upload speed of server is slow)
  * Edit the file /etc/openvpn/server.conf via `sudo nano /etc/openvpn/server.conf`.  
  * Comment out `push "redirect-gateway def1"`.
  * Close and save the file with Ctrl+X, enter y, enter.
  * Restart OpenVPN via `sudo systemctl restart openvpn@server.service`

## 6. DNS-over-HTTPS [Optional]

* Copy these commands to terminal.

```bash
cd ~
wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.tgz
mkdir argo-tunnel
tar -xvzf cloudflared-stable-linux-arm.tgz -C ./argo-tunnel
rm cloudflared-stable-linux-arm.tgz
sudo cp ./argo-tunnel/cloudflared /usr/local/bin/
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

## 7. Setup Pihole to run with DNS-over-HTTPS

* Create an additional DNSMasq configuration file: `sudo nano /etc/dnsmasq.d/02-dnscrypt.conf`.
* Enter the following in the file: `server=127.0.0.1#54`.
* If you did section 6, you have to do this one.
* Edit 01-pihole.conf: `sudo nano /etc/dnsmasq.d/01-pihole.conf`.
* Comment (#) out all server references, which means everything which looks like: `#server=...`
* Edit setupVars.conf: `sudo nano /etc/pihole/setupVars.conf`.
* Comment (#) out all piholeDNS references: `#PIHOLE_DNS_=...`.
* Restart DNSMasq: `sudo systemctl restart dnsmasq`.
* Reboot Raspberry Pi the last time with: `sudo reboot`.

## Warning Pihole 4.0 and up

If you want to show block page, follow this configuration [https://docs.pi-hole.net/ftldns/blockingmode/](https://docs.pi-hole.net/ftldns/blockingmode/)

## 8. Pihole 5.0 and up Export Blocklist to use anywhere.

* Go to /etc/pihole
* Run 
```
sqlite3 gravity.db "select domain from gravity
union
select domain from domainlist where type = 1
except
select domain from domainlist where type = 0;" > mybiglist.txt
```
* Then host `mybiglist.txt` in anywhere to use for Adguard or Blockada for Android
