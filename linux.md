# Notatie
Parameters:
* `<param>`: te vervangen door parameter `param`
* `{a|b|c}`: te vervangen door één van de opties `a`, `b` of `c`
* `[param]`: optionele parameter `param`

Adressen:
* `ip_host`: een IP-**hostadres** (bv. 192.168.1.1)
* `ip_addr`: een IP-**hostadres+mask in CIDR-notatie** (bv. 192.168.1.1/24)
* `ip_prefix`: een IP-**netwerkadres in CIDR-notatie** (bv. 192.168.1.0/24)

*De meeste commando's vereisen uiteraard root/sudo - use your common sense & RTFE!!!*
# Help/documentatie
* `<command> --help` of (soms) `<command> -h`: beknopt overzicht voor `command`
* `man <command>`: hulp over commando `command`
* `man <filename>`: hulp over configuratiebestand `filename`
* `man -k <keyword>`: zoeken naar onderwerp `keyword`

## Navigeren in man-pages
* zoeken: `/word` zoekt naar `word` (bevestigen met Enter)
* `n`: naar volgend zoekresultaat
* `G`: naar einde
* `q`: afsluiten (quit)

# Services (daemons)
* Beheer: `systemctl {start|stop|reload|restart|status} <service>`
   * niet elke service ondersteunt bv. `reload` --> RTFE!
* Autostart: `systemctl {enable|disable} <service>`
* Problemen?
  * check de status van de service: `systemctl status <service>`!
  * Netwerkservice?
  check ook of er effectief iets luistert op de verwachte poort:
  bv. `ss -ltn`
    * `-l`: enkel open (*listening*) ports
    * `-t`: enkel TCP (UDP: `-u`)
    * `-n`: poortnummers i.p.v. namen uit `/etc/services`
    * `-4`, `-6`: enkel IPv4/v6
    * Dual-stack (IPv4+IPv6) wordt getoond als IPv6!
  * raadpleeg de log van `systemd` met `journalctl -xe`
    * `-x`: extra uitleg tonen waar mogelijk
    * `-e`: spring meteen naar het einde
  * logfiles: te vinden in `/var/log`
    * `/var/log/messages`, `/var/log/syslog`: algemene logs
    * veel programma's hebben eigen log --> `ls -l /var/log`

# IP Configuratie
Configuratie tonen: `ip addr` (`ifconfig` is achterhaald!)
```bash
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
   valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
   valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
link/ether 00:0c:29:92:cd:d7 brd ff:ff:ff:ff:ff:ff
inet 192.168.23.129/24 brd 192.168.23.255 scope global ens33
   valid_lft forever preferred_lft forever
inet6 fe80::20c:29ff:fe92:cdd7/64 scope link
   valid_lft forever preferred_lft forever
```
  * `lo` is de loopback interface (127.0.0.1), dus **nooit** de juiste!
  * `LOWER_UP` --> kabel OK (layer 1)
  * `UP` --> layer 2 OK
  * `state DOWN` verder op de regel: interface disabled --> `ip link set <iface> up`
  * `NO-CARRIER` --> interface up maar kabel niet (goed) aangesloten
  
## Tijdelijke configuratie: weg na reboot

* `<iface>` := interface (netwerkkaart) --> zie `ip link` voor lijst, bv.: `eth0`, `wlan0`, `eno1`, `ens33`, `ens0p8`, ...
* Enable interface: `ip link set <iface> up`
* DHCP: `dhclient <iface>`
* Static IP: `ip addr add <ip_addr> dev <iface>`
* Weer verwijderen: `ip addr del <ip_addr> dev <iface>`

## Permanent: *altijd in configuratiebestand!*

### Debian/Traditional Linux
Configuratiebestand: `/etc/network/interfaces`
```bash
auto <iface>              # automatisch up bij opstarten

iface <iface> inet dhcp   # dynamisch adres

iface <iface> inet static # statisch
 address <ip_addr>
 gateway <ip_host>          # default gateway niet vergeten!
  ```
* Zie `man interfaces` voor meer syntax/opties!
* Nieuwe configuratie toepassen: `systemctl restart networking` of `ip link set <iface> down && ip link set <iface> up`
* DNS: servers toevoegen/aanpassen in `\etc\resolv.conf`

### Raspbian (dhcpcd)

**Niets veranderen in `/etc/network/interfaces`, anders doe je dhcpcd kapot! Ook DNS, ... moet via `dhcpcd.conf`**

Bestand: `/etc/dhcpcd/dhcpcd.conf`
* zie commentaar in het bestand zelf voor voorbeelden
* `man dhcpcd.conf` voor meer syntax/opties

```bash
# dynamisch: geen verdere config nodig

# static configuration:
interface <iface>
 static ip_address=<ip_addr>
 static routers=<ip_host>   # default gw!
 # static IP --> ook altijd static DNS!
 static domain_name_servers=<ip_host> # meerdere scheiden door spaties

# fallback profile
# enkel actief in geval DHCP faalt
# (zoals "Alternate configuration" in Windows)
profile <profile_name>
 static ip_address=<ip_addr>
 # ... verdere opties zoals bij static config

interface <iface>
 fallback <profile_name>
```
## Routing
* routetabel tonen: `ip route`
* check: 
  * 1 default gw
  * 1 route naar elk verbonden netwerk (`scope link`)
  * geen 'zombie' routes (bv. naar oud subnet na wijziging IP)
* default gateway toevoegen/verwijderen: `ip route {add|del} default via <ip_host> dev <iface>` (tijdelijk: permanent zie IP configuratie!)

# Wireless (wpa_supplicant)
## Open netwerk
Verbinden: `iw <iface> connect <SSID>`

## WPA: `wpa_supplicant`
### Configuratiebestand: `/etc/wpa_supplicant/wpa_supplicant.conf`

* zie `man wpa_supplicant.conf` voor meer opties & voorbeelden
  * **let op: service `wpa_supplicant` vs. configuratie `wpa_supplicant.conf` - allebei hebben een man-page!**
* nog meer voorbeelden in `/usr/share/doc/wpa_supplicant/examples`
* genereer een kant-en-klaar netwerkblok voor WPA-PSK (personal) met `wpa_passphrase '<SSID>' '<P@ssw0rd>'` (quotes zijn nodig in geval van spaties en/of speciale karakters!)
```bash
  # WPA-PSK (a.k.a. WPA-Personal)
  network={
    ssid="<SSID>"
    psk="<passphrase>"    # plain text
    psk=<hexstring>       # hex-encoded (bv. door wpa_passphrase: zonder quotes!)
  }

  # WPA-Enterprise (kan je zelf niet configureren op bv. Linksys --> enkel voor Howest/bedrijfsnetwerk)
  network={
    ssid="<SSID>"         # bv. "HowestWireless"
    key_mgmt=WPA-EAP  
    eap=PEAP
    pairwise=CCMP
    identitity="<email>"
    password=hash:<hash>
  }

  # Overige opties
  network={
    priority=<int>        # volgorde voorkeur voor netwerken
    scan_ssid=1           # voor hidden networks
    id_str="<string>"     # zelfgekozen naam voor netwerk
  }
```

Wachtwoord hashen (enkel voor WPA-Enterprise): `echo -n "P@ssw0rd" | iconv -t utf16le | openssl md4`

**!!!Let op voor wachtwoorden in command history!!!**
* `history`: weergeven
* `history -d <line number>`: enkel 1 regel verwijderen
* `history -c`: nuke it all

### Tool: `wpa_cli`
* zeker juiste interface kiezen! (default is fout, moet `wlan0` zijn)
  * dus starten met `wpa_cli -i wlan0` of indien vergeten: `interface wlan0` in de CLI zelf
* uitvoerbaar zonder `sudo` als je lid bent van de groep `netdev`
* je kan i.p.v. de interactieve modus ook gewoon een commando meegeven als parameter, bv. `wpa_cli -i wlan0 status`

#### Belangrijke commando's voor `wpa_cli`:
* `interface <iface>`: juiste interface selecteren (!!!)
* `reconfigure`: configuratiebestand opnieuw inlezen
* `list_networks`: geconfigureerde netwerken tonen
  * `TEMP_DISABLED`: waarschijnlijk fout wachtwoord
* `select_network <nummer>`: probeer verbinden met netwerk (zie `list_networks` voor nummers)
* `scan`: zoeken naar netwerken
* `scan_results`: om resultaat te tonen
* `disconnect`: om verbinding te verbreken
* `status`: verbindingsstatus
* `terminate`: wpa_supplicant service stoppen

# Troubleshooting
## Algemeen
* `ping <default gw>`: Lokaal netwerk OK
* `ping 8.8.8.8`: Extern netwerk OK (werkt niet op Howest net)
* `nslookup google.com`: DNS OK (package `dnsutils` vereist)
* `wget google.com`: Internet werkt

## Wireless
### `wpa_supplicant`
* check of de service draait: `ps aux | grep wpa`
* zo niet --> wellicht fout in Configuratiebestand
  * probeer manueel te starten: `sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf`
  * hopelijk zie je hier in meer detail wat er mis is in configuratiebestand
  * stop met `<ctrl+C>`, fix error, retry
  * indien succesvol --> stoppen en normaal starten
* hoewel er een `wpa_supplicant` is via `systemctl`, is dat **niet de juiste**!
  * de juiste wordt gestart door `dhcpcd` --> herstart die en kijk of `wpa_supplicant` mee opstart
* check `wpa_cli -i wlan0 status`, `wpa_cli -i wlan0 list_networks`

### Misc
* verbinding valt weg na verloop van tijd?
  * check `iw <iface> get power_save`
  * zet uit met `iw <iface> set power_save off`
* vliegtuigmodus --> staat soms plots overklaarbaar aan:
  * check met `rfkill list`
  * uitzetten met `rfkill unblock <iface>`

# Firewall
ufw [--dry-run] enable|disable|reload

ufw [--dry-run] default allow|deny|reject [incoming|outgoing|routed]

ufw [--dry-run] logging on|off|LEVEL

ufw [--dry-run] reset

ufw [--dry-run] status [verbose|numbered]

ufw [--dry-run] show REPORT

ufw  [--dry-run] [delete] [insert NUM] allow|deny|reject|limit [in|out] [log|log-all] [ PORT[/PROTOCOL] | APPNAME ] [comment COMMENT]

ufw [--dry-run] [rule] [delete] [insert NUM] allow|deny|reject|limit [in|out [on INTERFACE]] [log|log-all] [proto  PROTO‐
COL] [from ADDRESS [port PORT | app APPNAME ]] [to ADDRESS [port PORT | app APPNAME ]] [comment COMMENT]

ufw  [--dry-run] route [delete] [insert NUM] allow|deny|reject|limit [in|out on INTERFACE] [log|log-all] [proto PROTOCOL]
[from ADDRESS [port PORT | app APPNAME]] [to ADDRESS [port PORT | app APPNAME]] [comment COMMENT]

ufw [--dry-run] delete NUM

ufw [--dry-run] app list|info|default|update

  
# Software
## Package manager: `apt`
* vooraf **ALTIJD** `apt update` --> pakket database synchroniseren met Internet
* `apt <command> <package>`: `commando` toepassen op `package` (meerdere scheiden met spatie)
  * `install`: installeren
  * `remove`: verwijderen, configuratie bijhouden
  * `purge`: verwijderen incl. config
  * `show`: info tonen
  * `-y`: ja, ik ben zeker!
* Updates installeren: `apt <command>`
  * `upgrade`: updates installeren, niets verwijderen
  * `full-upgrade`: updates & overbodige verwijderen
  * `autoremove`: overbodige verwijderen, ook als optie mogelijk: `--auto-remove`

Start een upgrade best in een `screen` om problemen te vermijden als je SSH-verbinding wordt onderbroken

## Low-level: `dpkg`
  * `dpkg --list`: alle geïnstalleerde pakketten oplijsten
  * `dpkg -i <package.deb>`: pakket installeren uit `*.deb` bestand

# Gebruikers, groepen, rechten
## Gebruikers
* `whoami`: naam huidige gebruiker tonen
* `su - <user>`: switch user (user==blank --> root)
* `adduser`: gebruiker toeveogen
* `passwd`: wachtwoord wijzigen
* `passwd <user>`: wachtwoord voor `user` resetten (enkel root)
* `groups <user>`: groepen voor `user` tonen (of huidige gebruiker indien niet opgegeven)
* `usermod -aG <group> <user>`: gebruiker toevoegen aan groep(en) (meerdere groepen: scheiden met komma's)
   * Belangrijke groepen: `sudo` (mag `sudo`), `netdev` (mag `wpa_cli`)
   * RPi-specifiek: `gpio`, `i2c`, `spi`

## Rechten
* `ls -l`: rechten & owner tonen voor bestanden

```bash
  pi@raspberry:~ $ ls -l
  total 102436
  lrwxrwxrwx  1 root root          4 Dec  9 17:41 run -> /run
  drwxr-xr-x  6 root root       4096 Dec  9 18:27 spool
  -rw-------  1 root root  104857600 Dec  9 17:44 swap
  |---------    ---- ----  |-------- grootte (bytes)
  |    |         |    |
  |    |         |    owner (group)
  |    |         owner (user)
  |    rechten: owner/group/others telkens Read-Write-eXecute
  type: d=directory, l=symbolic link, -=file, ...
```
* `chown <user>[:<group>] <file>`: wijzig eigenaar (user of user+group) van `file`
* `chgrp <group> <file>`: wijzig eigenaarsgroep van `file`
* `chmod <mode> <file>`: wijzig mode (**permissies**)
  * absoluut: bv. `chmod 644 <file>`: huidige rechten vervangen
  * relatief: bv. `chmod +x <file>`: huidige rechten behouden en eXecute toevoegen
  * octaal: 4 = Read, 2 = Write, 1 = eXecute
  * bv. `chmod 754 <file>`:
    * 7 = read + write + execute voor **user**
    * 5 = read + execute voor **groep**
    * 1 = enkel read voor **others** (de rest van de wereld)
	
### Scripts uitvoeren:
* eerst `chmod +x <script>`: execute permissie geven
* dan uitvoeren met vermelding expliciet pad (in huidige directory: `./script`)
	
# Tips & tricks
## Keyboard layout 
* Ofwel `dpkg-reconfigure keyboard-configuration`
* Ofwel rechtstreeks `/etc/defaults/keyboard` aanpassen:
```bash
XKBMODEL="pc105"
XKBLAYOUT="be"
XKBVARIANT=""
XKBOPTIONS=""

BACKSPACE="guess"
```
* Nieuwe config toepassen met `sudo setupcon`
* Voor de GUI:
  * `setxkbmap be`
  * `udevadm trigger –subsystem-match=input –action=change`

## Wachtwoord vergeten
* unplug power
* verwijder SD-kaart & steek in je PC
* voeg in `cmdline.txt` toe, **ALLES op dezelfde regel**: `rw init=/bin/bash`
* plaats SD, verbind HDMI & toetsenbord, start op

```bash
passwd <user>     # LET OP: querty/azerty!
sync
exec /sbin/init
```
* verwijder de toegevoegde items weer uit `/boot/cmdline.txt`
* reboot

## Kiosk mode
TODO