# C programozás Windows Subsystem for Linux segítségével

Ez a leírás röviden végigvezet a WSL beüzemelésén, a GCC fordító telepítésén és egy példa kód fordításán, futtatásán.

## Mi az a WSL?

A [Windows Subsystem for Linux][wsl_wiki] segítségével linuxos programokat futtathatunk windowsos gépünkön. Amennyiben Windows 10-es géped van és nem akarsz dual boottal, vagy virtuális gép telepítésével bajlódni, ez a legegyszerűbb módja annak, hogy a kabinethez hasonló környezetben a GCC fordítóval dolgozz.

## Követelmények

* 64 bites x86 alapú számítógép (10 évnél nem öregebb asztali gépek és laptopok zöme)
* 64 bites Windows 10 operációs rendszer 1607-es verzió vagy újabb (S változat nem támogatott)


# WSL engedélyezése

Ahhoz hogy használni tudjuk, engedélyeznünk kell a WSL-t. Ehhez indítsuk el adminisztrátori jogosultsággal a PowerShellt.

![PowerShell indítása][start_powershell]

Adjuk ki a következő parancsot:

    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux

Ezután újra kell indítanunk a gépet.


# Linux disztribúció telepítése

Indítsuk el a Microsoft Store-t, ahol többféle linux [disztribúció][linux-distro] közül választhatunk. Mivel a kabinetben Debian van, így ebben a leírásban is azt mutatjuk be. A többi kezelése hasonló, de több ponton eltérhet (pl új programok telepítése).

Keressünk hát rá a Debianra, telepítsük és indítsuk el!

![Debian a Microsoft Store-ban][ms_store_debian]

Első indításkor kicsit szöszölni fog, majd kér egy felhasználónevet és egy jelszót. Ennek nem kell megegyeznie a Windowsos loginunkkal, de valamit mindenképp meg kell adnunk. Ez egy teljesen külön felhasználó lesz, ami csak a WSL-en belül létezik. Mint később több esetben, itt sem fogjuk látni a begépelt jelszó karaktereket, még csillagokat sem a helyükön (nehogy valaki megszámolja >:)).

![Sikeres telepítés][debian_after_install]


# Fordítóprogram telepítése

Az így kapott környezet a legalapvetőbb eszközöket tartalmazza (pl a szokásos linux parancsok természetesen működnek), viszont fordítót nem.

A telepítéshez két parancs szükséges. Az első:

    sudo apt update
    
Ez frissíti a rendelkezésre álló programokat, azok verzióit. Minden telepítés előtt érdemes lefuttatni, hogy ne egy régebbi verziót telepítsunk. Első használatkor a [sudo][sudo_wiki] egy rövid, kissé talán misztérikus üzenetet ad, majd kéri a jelszavunkat. A sudo a linuxos megfelelője a 'futtatás rendszergazdaként' opciónak. A legtöbb program jobb, ha nem rendszergazdaként fut, így fontos fájlokhoz például nem fér hozzá, viszont a telepítésnek muszáj. Adjuk hát meg a telepítéskor megadott WSL felhasználónk jelszavát.

![Csomaglisták frissítése][apt_update]

Most már jöhet a tényleges telepítés:

    sudo apt install gcc
    
Az [APT][apt_wiki] ájékoztat, hogy pontosan milyen [csomagokat][package_manager_wiki] (programokat, programkönyvtárakat) fog telepíteni, és az mennyi helyet foglal majd a gépünkön. Írjunk be egy `y`-t és enter. 

![Települ a gcc][gcc_installing]

Szokás szerint onnan tudjuk, ha sikeresen lefutott a telepítés, ha visszakapjuk a promptot és nem írt ki előtte hibaüzenetet.


# A Windows fájlrendszerének elérése WSL-ből

A WSL nagy előnye, hogy nem kell nagyon kilépnünk a komfortzónánkból egy kis Linux-ízelítőért. A linuxos parancssorunkból a teljes külső fájlrendszert elérjük, így ott dolgozunk ahol akarunk, olyan grafikus szerkesztőt használunk, amilyet szeretnénk, elég csak fordítani és futtatni terminálból.

Emlékezzünk vissza a terminálos fájlkezelésénél tanult alapparancsokra:

* `pwd`: print working directory - aktuális könyvtár kiírása.
* `cd`: change directory - könyvtárváltás, egy almappa nevét megadva utána abba lépünk, `..`-tal pedig az eggyel fentebbibe.
* `ls`: list - listázás, kiírja az aktuális könyvtár tartalmát.
* `mkdir`: make directory - könytvárlétrehozás, értelem szerűen.
* `rmdir`: remove directory - *üres* könyvtár törlése.
* `rm`: remove - fájl törlése. `-rf` kapcsolóval viszont egy komplett mappát töröl a tartalmával együtt. A művelet *nem visszavonható*, a törölt állományok nem kerülnek a lomtárba (linuxon alapból nincs is lomtár)!

Alapból a `/home/felhasznalonev` könyvtárunkban vagyunk, ami üres. Itt is nyugodtan dolgozhatunk, azonban ennek a tartalmát kívülről, a Windows felületéről nehézkes elérni, ezért javaslom, hogy inkább használjunk kinti mappát. Lépjünk a `/mnt` mappába és írassuk ki a tartalmát részletesen az `ls -l` paranccsal!

![/mnt tartalma][mnt_contents]

Itt bizony a Windows-os meghajtóinknak megfelelő mappákat találunk, melyeket szabadon böngészhetünk. Válasszunk egy nekünk tetsző mappát, és navigáljunk bele a `cd` paranccsal!


# Első programunk

Hozzunk létre kedvenc Windowsos szövegszerkesztőnkkel (pl [Notepad++][npp_website], [Atom][atom_website], [Sublime][sublime_website]) egy új C forrásfájlt az imént választott mappában mondjuk `elso.c` néven. Írjunk bele valami teszt kódot, pl ezt:

```C
#include <stdio.h>

int main() {
    printf("Hello, world!\n");
    return 0;
}
```

Mentsük el és váltsunk vissza a terminálunkra. Adjuk ki újra az `ls` parancsot és a fájl meg kell hogy jelenjen. Ha nem így történt, ellenőrizzük az elérési utakat, biztos mindkét helyen ugyanott vagyunk-e.

Jöhet a fordítás, melegen ajánlom a `-Wall` kapcsoló használatát:

    gcc -Wall -o elso elso.c

Majd a futtatás:

    ./elso
    
És az eredmény remélhetőleg egy tetszetős köszönés félig Windows félig Linux masinánktól: `Hello, world!`.

![Fordul és fut!][compile_and_run]

[start_powershell]: https://i.imgur.com/AF5nM2G.png
[linux-distro]: https://hu.wikipedia.org/wiki/Linux-disztrib%C3%BAci%C3%B3
[ms_store_debian]: https://i.imgur.com/os787RN.png
[debian_after_install]: https://i.imgur.com/itzXEdn.png
[sudo_wiki]: https://hu.wikipedia.org/wiki/Sudo
[apt_update]: https://i.imgur.com/snt7aak.png
[apt_wiki]: https://hu.wikipedia.org/wiki/Advanced_Packaging_Tool
[package_manager_wiki]: https://en.wikipedia.org/wiki/Package_manager
[gcc_installing]: https://i.imgur.com/7eS2KDx.png
[mnt_contents]: https://i.imgur.com/qmLefis.png
[compile_and_run]: https://i.imgur.com/zIpYLar.png
[npp_website]: https://notepad-plus-plus.org/
[sublime_website]: https://www.sublimetext.com/
[atom_website]: https://atom.io/
[wsl_wiki]: https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux
