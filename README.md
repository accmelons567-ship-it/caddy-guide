# caddy-guide
This guide would explain how to have certs for your services at home using caddy. Also using dns-01 so you don't have to open any ports or mess with firewalls. I would be using my technitium as an example.

# Needs :

proxmox

duckdns domain

Technitium

devices using your technitium dns

# Technitium (skip if already installed)
If you have not already have a technitium lxc to make a technitium lxc it is very simple,

create ct any name you want and password, use debian 13 as the template. 1vcpu, 512mib ram, 8gib storage, static ip and gateway to your router ip, and any dns you want since it going get changed

Example:

Ip 192.168.86.10/24

gateway 192.168.86.1

Dns server 9.9.9.9

get into your ct console put root as username than your password you put

```
apt update && apt upgrade -y

apt install curl

curl -sSL https://download.technitium.com/dns/install.sh | bash
```

go to your ip:5380, the static ip you put for your ct for technitium.

Example: 192.168.86.10:5380

set your home devices to that ip or go to your router and change the dhcp to give out the technitium ip.

Example: 192.168.86.10

if you want you can put forwarders instead of letting the technitium do recursive look ups by going into setting proxy & forwarders and pick any one you want then scroll on the bottom and save it.

# Duckdns
duckdns.org login with your account and add any domain

In technitium go into zones add your-domain.duckdns.org as a conditional forwarder zone and use this-server for the fwd and when you made it, go into the conditional forwarder zone you made and add A records like dns as the name and point it to your caddy ip address in

Example:

Name : dns

Type: A

Ipv4 address: 192.168.86.14

The point of this is so you dns get answered locally and you don’t need to go back to the internet just for the query. Also when it pop up the ip address for ipv4 you can just put your caddy ip address.

# Caddy
make a ct with a name, password debian 13 as template. With 1vcpu, 512 mib ram, 8gib storage a static ip and gateway to your router and technitium ip as dns server

Example:

192.168.86.10 as my dns server

192.168.86.14/24 is my static ip for caddy

192.168.86.1 is gateway
```
apt install -y debian-keyring debian-archive-keyring apt-transport-https curl

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list

chmod o+r /usr/share/keyrings/caddy-stable-archive-keyring.gpg

chmod o+r /etc/apt/sources.list.d/caddy-stable.list

apt update

apt install caddy
```
This installed the default caddy with basic modules and setup systemd for you

# Caddy modules
```
systemctl stop caddy

curl -o caddy 'https://caddyserver.com/api/download?os=linux&arch=amd64&p=github.com%2Fcaddy-dns%2Fduckdns'

chmod +x caddy

./caddy list-modules
```
Make sure it shows the duckdns module

This installs the custom modules and if you want other modules go caddyserver.com/download and select the plugins you want and copy the address of the download button and replace ' ' with it instead.

# Moving
```
dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy

mv ./caddy /usr/bin/caddy.custom

update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10

update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom 50
```
This moves the custom as the main and the default as the backup and to update caddy you do 
```
caddy upgrade
```
More info below

https://caddyserver.com/docs/build#package-support-files-for-custom-builds-for-debianubunturaspbian

# TOKEN
```
nano /etc/caddy/caddy.env

DUCKDNS_API_TOKEN=your-actual-token-here

chmod 600 /etc/caddy/caddy.env

systemctl edit caddy

[Service]

EnvironmentFile=/etc/caddy/caddy.env

Put this between the comment lines

It should say

### anything between here and the comment below will be contents

(Paste here)

### edits below this comment will be discarded

systemctl daemon-reload
```
Putting the api key in env file make it safer than just putting it in the raw caddyfile

# Caddyfile
```
nano /etc/caddy/Caddyfile
```
```
{

acme_dns duckdns {env.DUCKDNS_API_TOKEN}

}
```
This should be on the top of the caddyfile

Than add

```
dns.your-domain.duckdns.org {

reverse_proxy 192.168.86.10:5380
```
}

Than
```
systemctl restart caddy
```
```
journalctl -u caddy --no-pager | tail -20
```
Check if the cert got issued

If it did go on dns.you-domain.duckdns.org and it should bring you to your technitium page with secure connection and you can see the cert by clicking the icon next to the search bar.

If you don't know how to use nano

ctrl +x, y, enter

To save your edits and exit

replace my ip’s with yours
