# Checklist Linux Networking

1. [ ] Bepaal de juiste interface `<iface>`
   - *enkel* op Raspbian kan je er vanuit gaan dat de ingebouwde NIC `eth0` is en de ingebouwde WiFi `wlan0`
   - check anders de output van `ip link` of `ip addr` voor een overzicht 
2. [ ] Breng de interface up (check: `state UP` in de output van `ip link`)
   - [ ] DHCP of static?
      - [ ] test of DHCP werkt met `dhclient -d <iface>`
      - [ ] verzamel nodige gegevens voor static config:
		- [ ] (host-) IP-adres 
		- [ ] subnet mask OF CIDR prefix length (/xx)
		- [ ] default gateway voor alles buiten het lokale subnet
		- [ ] DNS servers voor naamresolutie
			- ofwel kan je het IP van de breedbandrouter gebruiken
		    - ofwel de DNS-servers die de router zélf mee kreeg: check pagina 'Status' op de webinterface
3. [ ] Debian of Raspbian?
    - [ ] Debian: `/etc/network/interfaces` voor IP en `/etc/resolv.conf` voor DNS
    - [ ] Raspbian: **enkel** `/etc/dhcpcd.conf` voor alle instellingen
4. [ ] Wijzig de configuratie:
   - DHCP 
	  - *Raspbian*: niets verder nodig, zou zo moeten werken 
	  - *Debian*: 
	    - voeg toe onderaan `/etc/network/interfaces`:
			```bash
			auto <iface>
			iface <iface> inet dhcp
			```
   - Static 
	  - *Raspbian*: wijzig `/etc/dhcpcd.conf` (nodige staat er al in commentaar normaal):
		```bash
		interface <iface>
		static ip_address=<ip_addr>  	# prefixnotatie met /xx!
		static routers=<ip_host>		# default gw
		static domain_name_servers=<ip_host> [ip_host]
		```		 
	  - *Debian*: voeg toe onderaan `/etc/network/interfaces`:
		```bash
		auto <iface>
		iface <iface> inet statuic
		```
5. [ ] Activeer nieuwe config:
    - [ ] `systemctl restart networking.service`
	- [ ] `ip link set <iface> down && ip link set <iface> up`
  - soms moet je allebei proberen voor het werkt
6. [ ] Wireless?
  - `/etc/wpa_supplicant/wpa_supplicant.conf` voor configuratie en `wpa_cli` voor beheer
  - gebruik wpa_passphrase om netwerkconfig te genereren voor WPA-personal:
  ```console
  # sudo -i
  $ wpa_passphrase <SSID> <passphrase> >> /etc/wpa_supplicant/wpa_supplicant.conf
  $ logout
  # wpa_cli -i wlan0
  > reconfigure
  > list_networks
  > select_network <#>
  > status
  > quit
  ```
  
- makkelijker op deze manier: met `sudo -i` 
- let op dubbele `>>` in commando of je overschrijft heel het bestand!!!
- juiste interface niet vergeten meegeven aan `wpa_cli`
- voeg `priority=1` toe aan netwerkblokje om het voorrang te geven t.o.v. NMCT-RPi