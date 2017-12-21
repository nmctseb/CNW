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