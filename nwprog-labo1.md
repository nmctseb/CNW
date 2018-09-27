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

* Pluralsight (video): [Introduction to the Bash Shell](https://app.pluralsight.com/library/courses/introduction-bash-shell-linux-mac-os) - *do the "learning check" to test yourself!*

## Syllabus

### Guides & tutorials

* Pluralsight (video): [Shell Scripting with Bash](https://app.pluralsight.com/library/courses/bash-shell-scripting)
* [The bash guide](http://mywiki.wooledge.org/BashGuide) - [new version (under construction)](https://guide.bash.academy/)
* [Bash beginner's guide](http://tldp.org/LDP/Bash-Beginners-Guide/html/)
* More at <http://wiki.bash-hackers.org/scripting/tutoriallist>

### Code style

* [Bash Pitfalls](http://bash.cumulonim.biz/BashPitfalls.html)
* [oVirt style guide (geared more towards sysadmins\)](https://www.ovirt.org/develop/infra/infra-bash-style-guide/)
* [Google's bash style guide](https://google.github.io/styleguide/shell.xml) - It's Google, you'll have to comply anyway
* <http://mywiki.wooledge.org/BashGuide/Practices>

### Exercises

- maak een functie die verifieert of de uitvoerende gebruiker `root` is 
- maak een functie die verifieert dat de uitvoerende gebruiker **niet**`root` is, maar wel `sudo` kan uitvoeren

- maak een alias voor het commando `sudo` die 
  - checkt of het 1e argument nano is 
  - zo ja, check of effectief root-rechten nodig zijn voor het gevraagde bestand
  - zo ja, doe het gevraagde, zo nee sluit af met een foutboodschap 
  
- maak een script `userclone` dat 2 argumenten heeft 
  - het eerste argument moet een bestaande gebruiker zijn, anders error&abort
  - het tweede argument mag **geen** bestaande gebruikersnaam zijnm anders error&abort
  - het script maakt een nieuwe user aan met deze naam 
  - maakt hem lid van al dezelfde groepen als de bestaande gebruiker (behalve zijn persoonlijke uiteraard)
  - stelt een tijdelijk wachtwoord 'NMCT' in, dat bij de eerste login moet veranderd worden 


#### More 
* <http://tldp.org/LDP/abs/html/scriptanalysis.html>
* <https://www.shellscript.sh/exercises.html>

## Further reading

* [Writing robust shell scripts](https://www.davidpashley.com/articles/writing-robust-shell-scripts/#id2382181)
* [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/index.html) - classic, but not for the faint of heart!

## Resources

* [GNU bash reference manual](https://www.gnu.org/software/bash/manual/bash.html)
