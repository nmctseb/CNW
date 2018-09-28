# Labo 1: Bash Shell Scripting

## Tools

* Use [Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) to take notes
  * in-line `commands` between \`backticks\`
  * blocks with \```lang ... ```
  * apply syntax highlighting!
    * for scripts: lang=bash
    ```bash
    #! /usr/bin/env bash
    wtf(){
      shit="magic"
      echo "it's fucking ${shit}"
    }
    ```
    * for copied console sessions: lang=console
    ```console
    me@my-box:~ $ wtf
    it's fucking magic
    ```

* Use a sensible editor!
  * don't like `vim`, `emacs`? Set up VSCode w/ extensions BASH-IDE + SSH-FS

## Prerequisites

* Pluralsight (video): [Introduction to the Bash Shell](https://app.pluralsight.com/library/courses/introduction-bash-shell-linux-mac-os) * *do the "learning check" to test yourself!*
* [GNU/Linux Command-Line Tools Summary](http://tldp.org/LDP/GNU-Linux-Tools-Summary/html/GNU-Linux-Tools-Summary.html)

## Syllabus

### Mandatory

* [The bash guide](http://mywiki.wooledge.org/BashGuide) - [all on one page](http://mywiki.wooledge.org/FullBashGuide)
* [Bash Pitfalls](http://bash.cumulonim.biz/BashPitfalls.html)
* [oVirt style guide (geared more towards sysadmins\)](https://www.ovirt.org/develop/infra/infra-bash-style-guide/)

### Further reading (optional)

* Pluralsight (video): [Shell Scripting with Bash](https://app.pluralsight.com/library/courses/bash-shell-scripting)
* [Bash beginner's guide](http://tldp.org/LDP/Bash-Beginners-Guide/html/)
* More at <http://wiki.bash-hackers.org/scripting/tutoriallist>
* [Google's bash style guide](https://google.github.io/styleguide/shell.xml
* [Writing robust shell scripts](https://www.davidpashley.com/articles/writing-robust-shell-scripts/#id2382181)
* [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/index.html) * classic, but not for the faint of heart!

## Resources

* [GNU bash reference manual](https://www.gnu.org/software/bash/manual/bash.html)

## Exercises

### Definitions

* POSIX
* {interactive,login,restricted} shell
* prompt, $PS{1..3}
* built-ins
* metacharacters
* globbing
* `noclobber`
* EOF
* signal, trap

### Interactive bash use

* zoek op hoe je het vorige commando opnieuw kan uitvoeren, maar dan met een ander programma (m.a.w. herhaal vorige regel maar vervang 1e woord)

### Commands (~= oneliners)

* het laatste veld van elke regel in `/etc/passwd` is de login-shell van de gebruiker. geef weer hoe vaak elke shell voorkomt.
* bereken de som van alle user id's in op het systeem
* tel het aantal systeemvariabelen (`env`) dat gedefinieerd is
* check of het commando werd uitgevoerd met `sudo`, zo ja druk de regel 'you're not really root :(' af
* check of je via SSH als root mag inloggen (=instelling PermitRootLogin in ssh conf), zo ja druk je een waarschuwing af dat dit onveilig is
* bepaal welk proces het meeste CPU-tijd heeft verbruikt en druk die tijd af samen met naam en PID van het proces
* bepaal hoe vaak elk soort bestand (commando `file`) voorkomt in `/bin`
* welk L4-protocol heeft het meeste poortnummers gedefineerd in `/etc/service/'? druk de winnaar af (tcp of udp) en het aantal voor beide
* maak een variabele `$n_cpu` met daarin het aantal CPU-cores in het systeem
* bepaal wat de meest recente python-versie versie is op https://www.python.org/ftp/python/; download die en pak het archief uit zonder een tijdelijk bestand op te slaan

### Functions (few lines or re-usable bits)

* maak een functie die verifieert of de uitvoerende gebruiker `root` is
* maak een functie die verifieert dat de uitvoerende gebruiker **niet** `root` is, maar wel `sudo` kan uitvoeren
* maak een alias voor het commando `sudo` die
  * checkt of het 1e argument nano is
  * zo ja, check of effectief root-rechten nodig zijn voor het gevraagde bestand
  * zo ja, doe het gevraagde, zo nee sluit af met een foutboodschap

### Scripts (complete tasks)

* maak een script `userclone` dat 2 argumenten heeft
  * het eerste argument moet een bestaande gebruiker zijn, anders error&abort
  * het tweede argument mag **geen** bestaande gebruikersnaam zijnm anders error&abort
  * het script maakt een nieuwe user aan met deze naam
  * maakt hem lid van al dezelfde groepen als de bestaande gebruiker (behalve zijn persoonlijke uiteraard)
  * stelt een tijdelijk wachtwoord 'NMCT' in, dat bij de eerste login moet veranderd worden

### More

* <http://tldp.org/LDP/abs/html/scriptanalysis.html>
* <https://www.shellscript.sh/exercises.html>
