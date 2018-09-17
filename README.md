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

Indítsuk el a Microsoft Store-t, ahol többféle linux [disztribúció][linux-distro] közül választhatunk. Mivel a kabinetben [Debian][debian_wiki] van, így ebben a leírásban is azt mutatjuk be. A többi kezelése hasonló, de több ponton eltérhet (pl új programok telepítése).

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
    
Az [APT][apt_wiki] ájékoztat, hogy pontosan milyen [csomagokat][package_manager_wiki] (programokat, programkönyvtárakat) fog telepíteni, és az mennyi helyet foglal majd a gépünkön. Üssünk be egy <kbd>y</kbd>-t és enter. 

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
* `rm`: remove - fájl törlése. `-rf` kapcsolóval viszont egy komplett mappát töröl a tartalmával együtt. A művelet **nem visszavonható**, a törölt állományok nem kerülnek a lomtárba (linuxon alapból nincs is lomtár)!

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

![Kód Notepad++-ban][code_npp]

Mentsük el, lehetőleg Unix [sortöréssel][newline_wiki] és [UTF-8 karakterkódolással][utf8_wiki]. Váltsunk vissza a terminálunkra. Adjuk ki újra az `ls` parancsot és a fájl meg kell hogy jelenjen. Ha nem így történt, ellenőrizzük az elérési utakat, biztos mindkét helyen ugyanott vagyunk-e.

> **Megjegyzés** A sortörés vagy új sor karakter operációs rendszertől függ. Manapság két elterjedt ábrázolása van: a [Unix][unix_wiki] féle `\n` és a Windows féle `\r\n`. Előbbit a legtöbb rendszer megérti, ezért általánosabbnak tekinthetjük, utóbbiban az extra `\r` megzavarhatja azokat az alkalmazásokat, akik nem számítanak rá (pl. furcsa dolgokat írnak ki a konzolra, vagy elszámolják egy szöveg hosszát). Amikor Windowson dolgozunk a legtöbb szerkesztő alapból a windowsos sortörést fogja használni, ez azonban általában átállítható.

> **Megjegyzés** A [karakterkódolás][codepage_wiki] azt adja meg, hogy mely betűket, karaktereket mely egész számmal reprezentáljuk a számítógép memóriájában, illetve a háttértárakon. Például a nagy `A` betű kódja szinte univerzálisan `65`. Az egyszerűbb karakterekkel, mint az írásjelek, angol ABC betűi, számok nem is szokott gond lenni, ezek elférnek a 7 bites [ASCII][ascii_wiki] táblában is, ami gyakorlatilag az összes többi kódolás alapját képezi.
> A gond a többi nyelv karaktereivel kezdődik, pl a magyar ékezetes betűk, vagy akár a teljesen különböző ciril, arab, héber, kínai, japán, koreai, stb. ábécék. Ezek a nyelvek tradícionálisan mind-mind különböző kódtáblát használtak, általában az ASCII-t kibővítre 8 bitre, ezáltal további 128 karaktert hozzáadva. A mai Windowsok is még midig egy ilyen nemzeti karakterkészletet használnak, magyar nyelv esetén a [Windows-1250][cp-1250_wiki]-est. Ha egy fájlt így mentünk el, és abban nem csak ASCII karakterek vannak, akkor az más rendszereken (Linux, macOS, más nyelvű Windows) rosszul fog megjelenni.
> Eme inkonzisztencia feloldására született meg az [Unicode][unicode_wiki], amely egy olyan kódtábla, melyben a világ valamennyi írásjelét ábrázolhatóvá akarják tenni. Az ilyen Unicode karaktereknek is sajnos többféle kódolása lehetséges, legelterjedtebb azonban az UTF-8, a trükkös helytakarékossága miatt (legalábbis elsősorban latin karaktereket tartalmazó szövegek esetén). A mai Linux rendszerek is az UTF-8 karakterkódolást használják alapból, ezért érdemes nekünk is abban menteni a fájljainkat.

Jöhet a fordítás, melegen ajánlom a `-Wall` kapcsoló használatát, hogy minél több potenciális hibát jelezzen a fordító:

    gcc -Wall -o elso elso.c

Majd a futtatás:

    ./elso
    
És az eredmény remélhetőleg egy tetszetős köszönés félig Windows félig Linux masinánktól: `Hello, world!`.

![Fordul és fut!][compile_and_run]

[start_powershell]: img/Screenshot_6.png
[ms_store_debian]: img/Screenshot_7.png
[debian_after_install]: img/Screenshot_9.png
[apt_update]: img/Screenshot_10.png
[gcc_installing]: img/Screenshot_11.png
[mnt_contents]: img/Screenshot_12.png
[code_npp]: img/Screenshot_14.png
[compile_and_run]: img/Screenshot_13.png

[apt_wiki]: https://hu.wikipedia.org/wiki/Advanced_Packaging_Tool
[ascii_wiki]: https://hu.wikipedia.org/wiki/ASCII
[atom_website]: https://atom.io/
[codepage_wiki]: https://hu.wikipedia.org/wiki/Karakterk%C3%B3dol%C3%A1s
[cp-1250_wiki]: https://en.wikipedia.org/wiki/Windows-1250
[debian_wiki]: https://hu.wikipedia.org/wiki/Debian
[linux-distro]: https://hu.wikipedia.org/wiki/Linux-disztrib%C3%BAci%C3%B3
[newline_wiki]: https://en.wikipedia.org/wiki/Newline
[npp_website]: https://notepad-plus-plus.org/
[package_manager_wiki]: https://en.wikipedia.org/wiki/Package_manager
[sublime_website]: https://www.sublimetext.com/
[sudo_wiki]: https://hu.wikipedia.org/wiki/Sudo
[unicode_wiki]: https://hu.wikipedia.org/wiki/Unicode
[unix_wiki]: https://hu.wikipedia.org/wiki/Unix
[utf8_wiki]: https://hu.wikipedia.org/wiki/UTF-8
[wsl_wiki]: https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux
