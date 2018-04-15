# Guild for Raspberrypi 3 to setup Openvpn + Pihole + DNS-over-HTTPS
## 1. Prerequisite
* Raspberry Pi 3 or 3+
* Internet
* Know how to use terminal and command lines
## 2. Install OS for Raspberry Pi
Read instruction at this page https://www.raspberrypi.org/help/noobs-setup/2/
## 2. Openvpn
I used PiVPN. Copy this command in terminal.
```bash
curl -L https://install.pivpn.io | bash
```
For step-by-step, watch this video https://www.youtube.com/watch?v=9RSHSt4RuLk
After finished, reboot Pi to let tun0 show up when setup pihole.
```bash
sudo reboot
```
## 3. Pihole
### For those who already installed pihole 
Edit /etc/pihole/setupVars.conf
```bash
sudo nano /etc/pihole/setupVars.conf
```
Change the line `PIHOLE_INTERFACE=eth0` to `PIHOLE_INTERFACE=tun0`

### For those who hasn't install pihole yet.
#### Copy this command in terminal.
```bash
curl -sSL https://install.pi-hole.net | bash
```
#### Choose an Interface
![Interface](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/interface.PNG?raw=true "Interface")

#### Choose DNS Provider
![DNS](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/dns.PNG?raw=true "DNS")

#### Choose Protocol
![Protocol](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/protocol.PNG?raw=true "Protocol")

#### Choose IP address
![IP address](https://github.com/quyentruong/Raspberrypi-3-Openvpn-Pihole-DNS-over-HTTPS/blob/master/docs/assets/image/IPaddress.PNG?raw=true "IP address")
Make sure IP address matches with your Pi and Gateway matches with your router.

### Web Admin Interface and log queries
You should choose `on` to easy manage logs.
## 4. DNS-over-HTTPS
