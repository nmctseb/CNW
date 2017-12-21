# Configuratie router
## Tab: Basic Setup
- *Local IP address*: het **interne** IP van de router **--> dus NIET het netwerkadres!!!**
- *Subnet mask*: duh
---
- *DHCP Server*: Enable
- *Starting DHCP address*: Geeft aan vanaf welk IP de DHCP server mag beginnen uitdelen --> als je addressen moet vrijhouden in het **begin**, moet je dat hier doen (*let op: als je het IP verandert, verspringt dit vanzelf weer!*)
- *Maximum Number of DHCP Users*: Bepaalt, samen met voorgaande, het EINDE van de adressen die de DHCP server uitdeelt --> moet je aanpassen om adressen op het **einde** van de range vrij te houden.

Voorbeeld: 
> Geef de router het laatste IP in netwerk 192.168.1.64/26. Hou 5 adressen vrij om statisch toe te kennen.

Oplossing:
1. Bereken subnet/adressen
> - Gegeven: *netwerkadres* 192.168.1.64/26
> - Subnet mask: /26 --> 255.255.255.192
> - Broadcast: 192.168.1.127
> - Bruikbare IP's: 192.168.1.65 t.e.m. 192.168.1.126
> - Eerste 5 vrijhouden --> .65 t.e.m. .70 --> DHCP vanaf .71

2. Configuratie router
> - **Local IP address**: opgave = laatste bruikbare IP = 192.168.1.126
> - **Subnet Mask**: 255.255.255.192
> - het *netwerkadres* hoef je nergens in te stellen! Het kan immers berekend worden uit IP en subnet mask.
> - 5 adressen vrijhouden --> DHCP Server:  **Starting IP address** = 192.168.1.71

## Status
- Hier zie je het **externe** IP van de router
- Alsook de DNS servers die de router (via DHCP) gekregen heeft op de externe interface
- Als het externe IP 0.0.0.0 is, is de internetverbinding niet OK en zul je dus nooit kunnen surfen op het netwerk achter de router!

# Port forwarding
## Opzet
De breedbandrouter vertaalt *uitgaande* netwerkverbindingen naar het (publieke IP) op de externe interface en stuurt ze door. Door de verbindingen bij te houden in het tabel, kan hij achteraf het antwoord opnieuw naar de juiste PC op het interne net sturen ([NAT](https://en.wikipedia.org/wiki/Network_address_translation)). 


*Inkomende* verbindingen zijn dus niet mogelijk: als er een ongevraagd pakket toekomt op de externe interface, weet de router immers niet naar waar het verder te sturen. Om een dienst (bv. Webserver) op het internet net toegankelijk te maken van buitenaf moeten we dit instellen op de router: *Port Forwarding*. 

## Port fowarding
Stel dat je een *intern* netwerk 192.168.1.0/24 hebt, achter een breedbandrouter met *extern* adres 120.143.23.56. Op het netwerk staat een Linux-PC met adres 192.168.1.5, waarop een webserver geïnstalleerd is die luistert op poort 80. Die wil je bereikbaar maken van buitenaf op poort 8080. 

Dat wordt dan (instellen bij *Applications & Gaming* --> *Single port forward*)
> - *Application*: HTTP
> - *External port*: 8080
> - *Internal port*: 80
> - *To IP address*: 192.168.1.5
> - *Enable*: Aangevinkt

Daarna kan je de website bereiken van buiten door naar het *externe* IP van je router te surfen. Een andere poort dan 80 moet je meegeven na een dubbelpunt achteraan het adres.
> http://120.143.23.56:8080