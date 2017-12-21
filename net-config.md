# Linux: Configuratie netwerk
## `ip`: (bijna)-alles-in-een netwerktool 
**`ifconfig` is achterhaald! Werkt bv. op Debian niet meer goed**
* `ip link`: interfaces tonen, up/down brengen, ...
* `ip addr`: IP adres tonen, toevoegen/verwijderen, ...
* `ip route`: routetabel tonen, **default gw tonen/instellen**, routes toevoegen/verwijderen (normaal niet nodig op clients)
*  ...en nog veel meer: `ip help` voor nog meer 'contexts'

Naast de man-page kan je argument `help` gebruiken, bv: `ip help`, `ip addr help`, `ip route help`... 

**Uitzondering: DHCP kan niet met `ip` --> `dhclient <iface>`**

Voorbeeld:
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
  * je ziet hier 2 interfaces: `lo` en `ens33`
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
# deze regel altijd:
auto <iface>              # automatisch up bij opstarten

# en dan ofwel...

# dynamisch adres
iface <iface> inet dhcp   

# statisch adres
iface <iface> inet static 
 address <ip_addr>		  # Je kan ofwel CIDR-notatie (met /xy achter het IP) gebruiken
 netmask <ip_mask>		  # ofwel de subnet mask instellen met deze regel
 gateway <ip_host>        # default gateway niet vergeten!
# verdere opties zoals network, broadcast zijn overbodig want kunnen berekend worden uit address + mask
```
* Blijf van de config voor de loopback `lo` af!
* Zie `man interfaces` voor meer syntax/opties
* Nieuwe configuratie toepassen: `systemctl restart networking` of `ip link set <iface> down && ip link set <iface> up` --> **soms allebei: trial and error**
* DNS: servers toevoegen/aanpassen in `\etc\resolv.conf`
* if all else fails: reboot

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

Wachtwoord hashen (enkel voor WPA-Enterprise): `echo -n 'P@ssw0rd' | iconv -t utf16le | openssl md4`

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

## DHCP
* met `dhclient -d <iface>` kan je testen of DHCP lukt (afsluiten met `Ctrl+C`)

## Wireless
### Hardware
- devices: `iw dev`
- interfaces: `iw phy`
- verbinding maken met **onbeveiligd** netwerk: `iw <iface> connect <SSID>`

### `wpa_supplicant`
* check of de service draait: `ps aux | grep wpa`
* zo niet --> wellicht fout in Configuratiebestand
  * probeer manueel te starten: `sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf`
  * hopelijk zie je hier in meer detail wat er mis is in configuratiebestand
  * stop met `<ctrl+C>`, fix error, retry
  * indien succesvol --> stoppen en normaal starten
* deze procedure kan je ook gebruiken als `wpa_cli reconfigure` lastig doet
  * stop dan eerst `wpa_supplicant` met `wpa_cli terminate`
  * hoewel er een `wpa_supplicant.service` is in `systemctl`, is dat **niet de juiste**!
  * de juiste wordt gestart door `dhcpcd` --> herstart die en kijk of `wpa_supplicant` mee opstart 
* check `wpa_cli -i wlan0 status`, `wpa_cli -i wlan0 list_networks`

### Misc
* verbinding valt weg na verloop van tijd?
  * check `iw <iface> get power_save`
  * zet uit met `iw <iface> set power_save off`
* vliegtuigmodus --> staat soms plots overklaarbaar aan:
  * check met `rfkill list`
  * uitzetten met `rfkill unblock <iface>`