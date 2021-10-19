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

* To connect to the instance, follow this [guide](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm)

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

## 6. Allow port in iptables
By default, the OS doesn't allow tun0 interface. We need tun0 to allow openvpn and pihole work. [More information](https://docs.pi-hole.net/guides/vpn/openvpn/firewall/)

```bash
sudo su - root
iptables -I INPUT -i tun0 -m comment --comment "# enable tun0 for pihole #" -j ACCEPT
iptables-save > /etc/pihole/rules.v4
```
## 7. Access web page on Internet [Optional]

Need to allow port 80 in iptables
```bash
iptables -I INPUT -i ens3 -p tcp --dport 80 -m comment --comment "# http #" -j ACCEPT
```

* In the Instance information, click on `subnet-xxx` -> `Default Security List xxx` -> `Add Ingrss Rules`
  * Source CIDR: 0.0.0.0/0
  * IP Protocol: TCP
  * Destination Port: 80

## 8. Setup DDNS [Optinal]
I use dynu.com. [IP Update Protocol](https://www.dynu.com/en-US/DynamicDNS/IP-Update-Protocol)

Password should be MD5/SHA-256 hash
```bash
wget "https://api.dynu.com/nic/update?hostname=example.com&password=098f6bcd4621d373cade4e832627b4f6" -O /dev/null
```

* Setup crontab to auto update IP
```bash
crontab -e
```
Choose nano. Then put at the end of the file
```bash
30 * * * * /usr/bin/wget "https://api.dynu.com/nic/update?hostname=example.com&password=098f6bcd4621d373cade4e832627b4f6" -O /dev/null
```
Press `Ctrl + x` and Y

## 9. Create LetsEncrypt for DDNS [Optional]

Setup acme for auto renew LetsEncrypt certificate.
```bash
git clone https://github.com/acmesh-official/acme.sh.git
cd ./acme.sh
./acme.sh --install -m your@email.com
```

Follow this [guide](https://www.dynu.com/resources/api/documentation) to get client_id and secret. Then replace in commands
```bash
export Dynu_ClientId="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
export Dynu_Secret="yyyyyyyyyyyyyyyyyyyyyyyyy"
acme.sh --issue --dns dns_dynu -d example.com -d www.example.com
```
[Previous - Raspberrypi 3 to setup Openvpn + Pihole + DNS-over-HTTPS](https://quyentruong.github.io/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/)
