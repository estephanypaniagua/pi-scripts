# pi-scripts
Pasos para permitir que un Raspberry Pi se conecte a un enrutador wifi conocido o genere automáticamente un punto de acceso de punto de acceso a Internet si no se encuentra dentro del rango de una red conocida.

Basado en [este tutorial][1].

## Requerimientos

``` bash
# Pasar al usuario root
sudo su 

# Instalar programas
apt install -y \ 
  hostapd \
  dnsmasq\ 
  iw

# Deshabilitar servicios
systemctl disable hostapd
systemctl disable dnsmasq
```

## Archivos a modificar

### Hostapd

#### `/etc/hostapd/hostapd.conf`

Archivo de configuración del access point.\
Es necesario modificar el ssid, el wpa_passphrase, y el country_code.

``` yml
#2.4GHz setup wifi 80211 b,g,n
interface=wlan0
driver=nl80211
ssid=REPLACE_MY_SSID
hw_mode=g
channel=8
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=REPLACE_MY_PASSWORD
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP TKIP
rsn_pairwise=CCMP

#80211n - Change PE to your WiFi country code
country_code=PE
ieee80211n=1
ieee80211d=1
```

#### `/etc/default/hostapd`

Archivo que permite leer la configuración declarada previamente.

``` yml
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### DnsMasq

#### `/etc/dnsmasq.conf`

Este archivo configura el rango que el DHCP utilizará en su access point, bajo el parámetro de `dhcp_range=IP_INICIO,IP_FIN,MASK,LIFE_SPAN` \
Añadir al final del archivo lo siguiente.

``` yml
#AutoHotspot config
interface=wlan0
bind-dynamic
server=8.8.8.8
domain-needed
bogus-priv
dhcp-range=10.10.10.150,10.10.10.200,255.255.255.0,12h
#Set up redirect for email.com
dhcp-option=3,10.10.10.10
address=/email.com/10.10.10.10
```

#### `/etc/dhcpcd.conf`

Añadir al final del archivo lo siguiente.

``` yml
nohook wpa_supplicant
```

### Creación del servicio

Crear un servicio que pueda ser manejado por systemctl en la ruta.

#### `/etc/systemd/system/autohotspot.service`

Copiar el contenido a este archivo

``` yml
[Unit]
Description=Automatically generates a Hotspot when a valid SSID is in range
After=multi-user.target
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/autohotspotN
[Install]
WantedBy=multi-user.target
```

Habilitar el servicio con 

``` bash
systemctl enable autohotspot.service
```

### Script de autohotspot

``` bash
# Descargar el archivo base
wget http://www.raspberryconnect.com/images/autohotspotN/autohotspotn-95-4/autohotspotN.txt

# Modificar la dirección IP con la nuestra
sed -i 's/192.168.50.5/10.10.10.10/' autohotspotN.txt

# Darle permisos de ejecución
mv autohotspotN.txt /usr/bin/autohotspotN
chmod +x /usr/bin/autohotspotN
```

### Configuración de wifi

#### `/etc/wpa_supplicant/wpa_supplicant.conf`

Añadir datos de la red a la cual nos queremos conectar

``` yml
network={
  ssid="REPLACE_MY_WIFI_SSID"
  psk="REPLACE_MY_WIFI_PASSWORD"
  key_mgmt=WPA-PSK
}
```

<!-- Enlaces -->
[1]: https://www.raspberryconnect.com/projects/65-raspberrypi-hotspot-accesspoints/157-raspberry-pi-auto-wifi-hotspot-switch-internet
