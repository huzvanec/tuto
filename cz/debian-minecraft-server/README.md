# 1. Stáhnutí Javy 21
Všechny moderní verze Minecraftu vyžadují Javu 21, [která je zatím v balíčkách pouze pro Debian testing a unstable](https://packages.debian.org/search?searchon=sourcenames&keywords=openjdk-21). Nejjednodušší je stáhnout [Oracle Javu jako deb package](https://www.oracle.com/java/technologies/downloads/#java21), ale třeba [tento tutoriál](https://computingforgeeks.com/install-java-jdk-or-openjdk-21-on-debian/) popisuje i jak sehnat OpenJDK.
# 2. Stáhnutí Minecraft server jaru
Minecraft serveru je třeba udělat složku a do ní je třeba stáhnout hlavní server jar.
### A. Oficiální Minecraft server.
#### Minecraft server napsaný a distribuovaný Mojang Studios.
\+ Jediné oficiálně podporované řešení<br>
\- Pomalejší (než ostatní řešení)<br>
\- Nepodporuje žádné modifikace<br>
**Stažení: https://www.minecraft.net/en-us/download/server**
### B. [PaperMC](https://papermc.io/software/paper)
#### Komunitou upravený a optimalizovaný Minecraft server, který podporuje pluginy.
\+ Optimizovaný, snaží se (alespoň trošičku) o využití více než jednoho threadu<br>
\+ Podpora vlastních i cizích pluginů<br>
\- Mojangem nepodporovaný<br>
\- Pomaleji se updateuje, adaptace nové Minecraft verze může trvat i několik týdnů od jejího vydání<br>
\- Některé bugfixy, které Paper zavedl mohou být nechtěné<br>
**Stažení: https://papermc.io/downloads/paper**
### C. Modovaný server
#### Komunitou upravený Minecraft server tak, aby podporoval mody.
Software, který umožňuje použití modů se nazývá mod loader. Mod loaderů je více, velké a používané mod loadery jsou v zásadě dva. Mody jsou psané vždy jen na jeden mod loader a nedají se mezi sebou míchat (např. mody pro Fabric nefungují na NeoForge atd.)
#### [Fabric Loader](https://fabricmc.net)
Stažení klient: https://fabricmc.net/use/installer/
Stažení server: https://fabricmc.net/use/server/
#### [NeoForge](https://neoforged.net/) (moderní nástupce [Forge](https://minecraftforge.net))
Stažení klient + server: https://projects.neoforged.net/neoforged/neoforge

Tento tutoriál se modům věnuje velmi málo a doporučuji si o nich nastudovat více předtím, než se do toho vrhnete.

# 3. Vytvoření startovacích bash scriptů
Do složky našeho Minecraft serveru je třeba udělat dva scripty, které budou startovat Minecraft server. Aby Minecraft server běžel i bez otevřeného terminálu je potřeba nainstalovat screen (`sudo apt install screen`). 

`start.sh` - script, který nastartuje Minecraft server
```bash
#!/bin/bash
JAR=<server.jar> # např. paper-1.21.1-57.jar
RAM=5G # Minecraft server by snad měl zvládnout běžet i na 1G paměti

java -Xms${RAM} -Xmx${RAM} -jar ${JAR} nogui
```

`screen-start.sh` - script, který pustí Minecraft server pod screen ("na pozadí")
```bash
#!/bin/bash

# -d -m    = nastartuje screen v odpojeném modu
# -S       = název screen procesu
screen -d -m -S "Minecraft Server" ./start.sh
```

Je třeba povolit scriptům spouštění.
```bash
chmod +x ./start.sh
chmod +x ./screen-start.sh
```

# 4. Potvrzení EULA
Je třeba potvrdit [Minecraft EULA](https://www.minecraft.net/en-us/eula).
```bash
echo 'eula=true' > eula.txt
```

# 5.  Testovací start
Test, jestli vše dobře funguje.
```bash
./start.sh
```
Pokud server nastartuje správně, v logu by se měla jako poslední zobrazit zpráva podobná téhle:
```
Done (12.340s)! For help, type "help"
```
Server jde zastavit příkazem `stop` nebo zkratkou `Ctrl+C`.

# 6. Bezpečnost
Server, který právě vznikl není bezpečný. Používá defaultní Minecraft port (25565) a připojit se k němu může úplně kdokoli.

První start serveru vygeneruje hromadu souborů včetně `minecraft.properties`, kde se nachází hlavní nastavení serveru. Zde je třeba změnit `server-port` na jinou hodnotu, aby Minecraft server nebyl tak lehce odhalitelný.

Taky je potřeba zapnout [whitelist](https://minecraft.wiki/w/Commands/whitelist). Whitelist omezí kteří hráči se na server mohou připojit.
Zapnutí whitelistu (v konzoli zapnutého Minecraft serveru)<br>
`whitelist on` - zapne whitelist<br>
`whitelist add <player name>` - přidá hráče na whitelist (jméno je case insensitive)<br>

# 7. Screen
Minecraft server pod screen se nastartuje pomocí `./screen-start.sh`.
Ke screen procesu se jde připojit pomocí `screen -r` (popř. `screen -r Minecraft Server` pokud běží screen procesů najednou více)
Od screen procesu se jde odpojit pomocí zkratky `Ctrl+A+D`.

# 8. Nějaké poznámky
### Není třeba se (hodně) bát používat neoficiální Minecraft server software.
Modovací i pluginová komunita je velká a skoro všechno je open source. Většina (větších i menších) Minecraft serverů dnes používá upravený Minecraft software (hlavně PaperMC).
### Jaký je rozdíl mezi pluginy a mody?
Obojí se programuje v Javě, obojí upravuje Minecraft. Jaký je rozdíl?
Pluginy upravují pouze kód Minecraft serveru, zatímco mody (skoro vždy) potřebují i modifikaci klienta. Pluginy mohou měnit fungování hry, ale nemohou "přidávat nové věci".
Např.
Pluginem nejdou přidat vlastní itemy, bloky, příšery atd. protože klient o nich nic netuší (na zobrazení potřebuje texturu, model...).
Pluginem jde upravit třeba jaký item vypadne z blocku, nebo co vypadne z příšery když umře. K tomu klient žádné další informace nepotřebuje.

### Jak povolit hráčovi používat příkazy
Pokud hráč chce používat příkazy (=commandy, cheaty), musí být operátor. To jde změnit (z konzole běžícího Minecraft serveru) příkazem `op <player name>` a `deop <player name>`. Operátorské oprávnění je ale hodně nebezpečné, (hráč) operátor má kontrolu prakticky nad celým serverem, má přístup ke všem příkazům jako Minecraft konzole (včetně whitelistu a dělání dalších operátorů). Pro podrobnější měnění oprávnění (např. přidělování jednotlivých příkazů hráčům) doporučuji mrknout na PaperMC plugin [LuckPerms](https://luckperms.net/).

### Dobré materiály
- [Minecraft Wiki: Jak si vytvořit vlastní server](https://minecraft.wiki/w/Tutorials/Setting_up_a_server)
- [Super playlist od super YouTubera jak začít s Minecraft pluginy (některé věci už jsou ale trochu starší)](https://www.youtube.com/watch?v=dem7dujCDvg&list=PLfu_Bpi_zcDNEKmR82hnbv9UxQ16nUBF7&index=3)
- [PaperMC plugin dev dokumentace](https://docs.papermc.io/paper/dev/project-setup)