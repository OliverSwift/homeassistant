# HA rig and Zigbee stuff

## The hub

### Mini PC
![BMAX](https://www.amazon.fr/dp/B09YTRWJQ)

### Coordinateur Zigbee
![Conbee II](https://www.amazon.fr/dp/B07PZ7ZHG5)

### Display Capacitive Touch
![8,8" IPS display 480x1920 60Hz Blacl frame and Touch](https://makerfabs.shop/products/makerfabs-8-8-inch-display-screen-1920-480-ips-screen-lcd-panel-hdmi-interface-driver-board)

## Zigbee Accessories

### Driver LD 12V
![Driver LED 220V vers 12V](https://www.amazon.fr/dp/B09LCDPWX)

### Bandeau LED 5m
![Bandeau 5m de LED 3000K 12V](https://www.amazon.fr/dp/B08CVJ2ST)

### Mi Light Dimmer
![Controleur Zigbee LED - MiBoxer @ Mi Light](https://fr.aliexpress.com/item/1005005124890685.html)

### Mesure de Puissance
![Tuya Power Meter](https://fr.aliexpress.com/item/1005004948381800.html)
Besoin d'un zha-quirks (`ts0601_din_power.py`)

## Home Assistant local server

### Installation
On Debian 11.
Need to build Python3.10.xx
```
# apt-get install build-essential gdb lcov pkg-config \
      libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
      libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
      lzma lzma-dev tk-dev uuid-dev zlib1g-dev
# wget 
# cd Python-3.10.11/
# ./configure --enable-optimizations
# make -j
# make altinstall
```

### configuration
Additional stuff
```
...
# Reverse Proxy
http:
  use_x_forwarded_for: true
  trusted_proxies: 127.0.0.1

# ZHA
zha:
  custom_quirks_path: /home/homeassistant/quirks
```

### Reverse Proxying from VPS
The idea is to create a ssh tunnel from home server to VPS. No incoming connection on home gateway.
Any connection on VPS's 9000 tcp port is forwarded to local HA server's usual 8123 port.
Need a non privileged account on VPS (ha with no login at all).
Install local ha server's homeassistant ssh public key on VPS.

Running `tunnel-ha.service` on local server sets up the tunnel.

Apache config on VPS:
```
<IfModule mod_ssl.c>
<VirtualHost *:443>
	ServerName ha.debon.org

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/ha

	ErrorLog ${APACHE_LOG_DIR}/ha_error.log
	CustomLog ${APACHE_LOG_DIR}/ha_access.log combined

	#Header always set Strict-Transport-Security "max-age=15552001; includeSubDomains;"

	RewriteEngine on
	RewriteCond %{HTTP:Upgrade} websocket [NC]
	RewriteCond %{HTTP:Connection} upgrade [NC]
	RewriteRule ^/?(.*) "ws://127.0.0.1:9000/$1" [P,L]

	ProxyPass / http://127.0.0.1:9000/
	ProxyPassReverse / "http://127.0.0.1:9000/
	ProxyRequests Off

SSLCertificateFile /etc/letsencrypt/live/debon.org/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/debon.org/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
```
