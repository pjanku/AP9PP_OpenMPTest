# Příklad různých způsobů paralelizace s využitím OpenMP a jejich důsledků
## Paralelizovaný problém
***
**Zadání:**
Vytvořte aplikaci, která provede velké množství operace incrementu (+1) a stejné množství operace decrementu (-1) 
nad proměnnou, která byla na počátku inicializována na hodnotu 0. Výsledná hodnota uložená v této proměnné 
by tedy měla být taktéž 0.

```c++
    const long long max = pow(10,8);
    long long sum = 0;
    for (long long i = 0 ; i < max;i++){
        sum++;
    }
    for (long long i = 0 ; i < max;i++){
        sum--;
    }
```

## Demonstrované přístupy paralelizace
***
Níže je uveden popis jednotlivých částí souboru [main.cpp](main.cpp). Příklady obsažené v tomto
souboru slouží k demonstraci možnosti využití některých paralelizačních přístupů a jejich dopad na 
výsledek a rychlost paralelizace. Jednotlivé výsleky měření času potřebného proběh příkladů se mohou 
zásadně měnit v závislosti na použitém HW.

**Nejedná se o vyčerpávající výčet možností, soubor možností typu "best practice" ani specifických 
návodů na provádění paralelizace**


### Využití paralelních sekcí

Při využití paralelních sekcí (kdy každá sekce řeší jednu smyčku), sdílené proměnné **BEZ** řízení přístupů k této proměnné je velké riziko vzniku
souběhu operací nad touto proměnnou. Takovéto riziko vede ve většině případů k **chybnému** výsledku operací.Zároveň
úroveň paralelizace je velmi nízka - pouze dvě vlákna pracují současně. Vzhledem k režii nutné pro 
vytváření a srávu vláken může docházet k zápornému zrychlení - tedy pomalejšímu běhu 
než v sekvenčním případě.

### Využití paralelních sekcí a atomických operací

Doplníme-li předchozí příklad o tzv. atomické* operace
dosáhneme tím správného výsledku výpočty. Negativním efektem bude ale sekvenční přístup k této proměnné,
kdy pouze jedno vlákno v jeden okamžik může provádět danou operaci. To společně s časem potřebným 
pro vykonání atomické operacene povede nejspíše k velkém zpomalení algoritmu.

### Využití paralelních sekcí a critických oblastí

Nahradíme-li v předchozím příkladu atomické operace kritickými sekcemi**, dojde ke stejnému efektu z pohledu výsledku 
operaco, tedy výsledek bude správný. Nicméně režije potřebná k zajištění těchto critických sekcí bude zásadně 
větší než režije potřebná k využití atomických operací. Výsledkem ted nejspíše bude zásadně pomalejší kód než 
v předchozích případech.

### Využití paralelních sekcí, vnořené paralelizace a operace redukce

Oproti předchozím případům přibyla vnořená paralelizace snažící se paralelizovat jednotlivé otočky daných cyklů. Tím se
v případě vícejádrového procesoru využije více paralelních vláken. 
Zároveň díky použití lokálních proměnných, není nutné pro každou operaci používat řízení přístupu a tak jsou tyto operace
zásadně rychlejší.

### Využití paralelizace smyčky for bez paralelních sekcí

Poslední příklad v daném souboru nevyužívá paralelních sekcí ale provádí paralelizace na úrovni jednotlivých smyček
for. Vzhledem k tomu, že za každou paralelní smyčkou for je implicitně zařazena bariéra (která v tomto případě je spíše 
kontraproduktivní), bylo využito klíčového slova **nowait** k jejímu potlačení. Na závěr je potřeba sjednotit dílčí
výsledky do požadované proměnné. To je provedeno v single sekci, před kterou je umístěna bariéra zajišťující
správné dokončení všech předchozích kroků. Vzhledem k tomu, že single sekce je vykonávána vždy pouze
jedním vláknem, nenní potřeba provádět žádné řízení přistupu k proměnné.

Tato varianta dává pro daný problém správný výsledek a to nejrychleji.

\* které zajišťují provedení v jednom kroku,

\*\* Kritické sekce mohou být vykonávány v jeden čas pouze jedním vláknem. 

---
## Závěr

Cílem těchto ukázek není udělit jednotný návod jak paralelilzovat algorimy, ale ukázat, že nevhodně zvolená
paralelizace může vést přesně k opačnému efektu. Výslednky paralelizace jsou mimo jiné velmi závislé na 
problému (algoritmu), který je paralelizován. V tomto konkrétním případě šlo o paralelizaci velmi jednoduché
a tedy i rychlé operace incrementu/decrementu. Z tohoto důvodu metody výcházely nejhůře přístupy, které mají 
jednorázově velkou režii na spouštění. Níže je uvedena tabulka s orientačními výsledky běhu
ukázkového programu na procesoru Intel Core i7 se čtyřmi fyzickými jádry a 2,3 GHz základní taktovací fekvence.


| **Implementace**                       | **Potřebný čas[s]** | **Výsledek** |
|----------------------------------------|---------------------|--------------|
| Sériový výpočet                        |                0.22 |            0 |
| Paralelní sekce (bez řízení přístupu)  |                0.47 |       691713 |
| Paralelní sekce (atomické operace)     |                3.13 |            0 |
| Paralelní sekce (kritické sekce)       |               24.11 |            0 |
| Paralelní sekce (vnořený paralelizmus) |                0.23 |            0 |
| Paralelní smyčky for                   |                0.09 |            0 |