# Poznámky k C++: třídy, dědičnost a práce s kolekcemi

Tento přehled shrnuje postupy používané při tvorbě menšího objektově orientovaného programu. Nejde o hotové řešení jedné konkrétní úlohy. Názvy jako `Zaklad`, `Potomek`, `historie` nebo `hodnota` jsou pouze obecné značky, které je potřeba nahradit podle tématu programu.

---

## 1. V jakém pořadí program stavět

Nejdřív vytvoř soubory a základní třídu. Potom přidej jednoho potomka a program přelož. Stejně pokračuj s druhým potomkem. Až samostatné objekty fungují, přidej společný vektor ukazatelů, virtuální volání a mazání paměti. Operátory a složitější zpracování kolekcí nech nakonec.

Po každé malé části použij ve Visual Studiu **Build → Build Solution** (`Ctrl+Shift+B`). Program bez chyb spusť přes `Ctrl+F5`. Když se objeví chyba, neopravuj deset věcí najednou; začni první chybou v seznamu.

---

## 2. Jak převést slovní požadavek na program

- Podstatná jména bývají názvy tříd nebo atributů.
- Slovesa bývají metody.
- „Text“ znamená obvykle `string`, celé číslo `int`, desetinné číslo `double`.
- „Seznam“ nebo „historie hodnot“ znamená obvykle `vector<typ>`.
- „Vrátí hodnotu“ znamená, že návratový typ není `void`.
- „Pouze vypíše“ nebo „provede změnu“ často znamená `void`.
- „Nemění objekt“ znamená `const` za závorkami metody.
- „Společné pro všechny objekty“ znamená `static`.
- „Každý potomek musí provést po svém“ znamená virtuální metodu; pokud základní třída nemá vlastní řešení, použije se `= 0`.

Příklad čtení věty: „Třída má chráněný textový název a seznam desetinných hodnot.“ Z toho plyne sekce `protected`, atribut typu `string` a atribut typu `vector<double>`.

---

## 3. Rozdělení souborů

Pro jednu třídu používej dvojici `Nazev.h` a `Nazev.cpp`. Spouštění a testování patří do `main.cpp`.

### Hlavička `.h` říká, co třída umí

Do hlavičky patří `#pragma once`, potřebné knihovny, definice třídy, atributy a hlavičky metod ukončené středníkem. Patří sem také slova `static`, `virtual`, `override`, `const`, `friend` a případně `= 0`.

### Implementace `.cpp` říká, jak to udělá

Na začátku připoj odpovídající hlavičku, například `#include "Zaklad.h"`. Před názvem metody se píše příslušnost ke třídě: `Zaklad::metoda`. Za hlavičkou metody už není středník, protože následuje tělo ve složených závorkách.

### `main.cpp` vše používá

Do `main.cpp` připojuj vlastní `.h` soubory, nikdy `.cpp`. Visual Studio přeloží zdrojové soubory vložené do projektu samo.

### Malý příklad rozdělení

V hlavičce může být pouze popis třídy:

```cpp
#pragma once
#include <string>

class Zaznam
{
private:
    std::string nazev;

public:
    Zaznam(const std::string& novyNazev);
    void vypis() const;
};
```

V implementaci jsou těla metod:

```cpp
#include "Zaznam.h"
#include <iostream>

Zaznam::Zaznam(const std::string& novyNazev)
    : nazev(novyNazev)
{
}

void Zaznam::vypis() const
{
    std::cout << nazev << std::endl;
}
```

V `main.cpp` stačí připojit hlavičku, vytvořit objekt a zavolat metodu. Důležité je všimnout si, že `Zaznam::` se píše v implementaci, ale ne uvnitř deklarace třídy.

---

## 4. Přístupy a základní třída

- `private`: člen používá jen daná třída.
- `protected`: člen používá daná třída i její potomci.
- `public`: člen lze volat také z `main`.

Společná data určená i potomkům se často dávají pod `protected`. Vlastní zvláštní údaj potomka bývá `private`. Konstruktory, gettery a ostatní volané metody patří pod `public`.

Základní třída obvykle obsahuje společné atributy, konstruktor, virtuální destruktor, gettery a virtuální metody. Nejdůležitější malé vzory jsou:

- konstruktor: `Zaklad(const string& nazev);`
- virtuální destruktor: `virtual ~Zaklad();`
- obyčejná virtuální metoda: `virtual void vypis() const;`
- čistě virtuální metoda: `virtual void analyzuj() const = 0;`

`= 0` udělá ze třídy abstraktní třídu. Takový objekt nelze vytvořit přímo, ale lze používat ukazatel `Zaklad*`, který ukazuje na potomka.

### Obecná kostra abstraktní třídy

```cpp
class Zaklad
{
protected:
    std::string nazev;
    std::vector<double> historie;

public:
    Zaklad(const std::string& nazev);
    virtual ~Zaklad();

    void pridejHodnotu(double hodnota);
    virtual void vypis() const;
    virtual void analyzuj() const = 0;
};
```

Toto není řešení konkrétního programu. Je to mapa: `nazev` a `historie` nahraď skutečnými atributy a `analyzuj` požadovanou činností.

### Konstruktor a inicializační seznam

V implementaci se atributy nastavují za dvojtečkou, například ve tvaru `Zaklad::Zaklad(...): atribut(parametr)`. Teprve potom následuje tělo konstruktoru. V materiálech se objevuje také zápis `this->atribut = parametr`; obě varianty fungují, ale inicializační seznam je pro konstruktory vhodnější.

---

## 5. `const`, reference a ukazatele

`const string& text` znamená: nepřeváděj celý text do kopie a uvnitř funkce ho neměň.

`void vypis() const` znamená: metoda nesmí měnit atributy právě používaného objektu. Hodí se pro výpisy, gettery, analýzy a porovnání. Nehodí se pro přidávání, mazání nebo jiné změny dat.

Co napíšeš v `.h`, musí být doslova stejné i v `.cpp`, včetně parametrů, `&` a koncového `const`.

Rozdíl při volání:

- běžný objekt používá tečku: `objekt.metoda()`;
- ukazatel používá šipku: `ukazatel->metoda()`;
- reference se uvnitř funkce chová jako běžný objekt a používá tečku.

Getter kolekce vracej referencí, pokud se má pracovat s původní kolekcí. Zápis `vector<double>&` dovolí její úpravu; `const vector<double>&` dovolí pouze čtení. Bez `&` by vznikla kopie.

### Tři různé návraty vektoru

```cpp
std::vector<double> getData();              // vrátí kopii
std::vector<double>& getData();             // dovolí měnit originál
const std::vector<double>& getData() const; // jen čtení originálu
```

Pokud funkce maže prvky z původního vektoru, potřebuje druhou variantu. Pokud pouze počítá nebo vypisuje, je vhodná třetí varianta.

---

## 6. Statický čítač objektů

Statický atribut patří celé třídě a existuje pouze jednou. Uvnitř třídy se pouze deklaruje ve tvaru `static int citac;`. V jednom `.cpp` se musí vytvořit a nastavit: `int Zaklad::citac = 0;`.

Postup čítače:

1. konstruktor provede `citac++`,
2. destruktor provede `citac--`,
3. statická metoda vrátí aktuální hodnotu,
4. volá se přes název třídy, například `Zaklad::getPocet()`.

Pozor na závorky: `static int citac();` není proměnná, ale deklarace funkce.

Dobrá kontrola je vytvořit dva lokální objekty uvnitř samostatného bloku `{ ... }`. Čítač má před blokem hodnotu 0, uvnitř 2 a po opuštění bloku znovu 0.

### Zápis čítače na třech místech

V těle třídy:

```cpp
private:
    static int citac;

public:
    static int getPocet();
```

V `.cpp` mimo všechny metody:

```cpp
int Zaklad::citac = 0;
```

V konstruktoru a destruktoru:

```cpp
Zaklad::Zaklad(const std::string& nazev)
    : nazev(nazev)
{
    citac++;
}

Zaklad::~Zaklad()
{
    citac--;
}
```

---

## 7. Odvozená třída

Veřejná dědičnost se zapisuje jako `class Potomek : public Zaklad`.

Potomek obvykle přidává vlastní privátní atribut. Jeho konstruktor musí přijmout údaje pro rodiče i vlastní údaj. Za dvojtečkou nejprve zavolá konstruktor rodiče a potom nastaví vlastní atribut: slovně tedy „společnou část pošli nahoru, zvláštní část si ulož“.

Metoda přepisující virtuální metodu má na konci `override`. Její název, parametry, návratový typ a případné `const` musí odpovídat rodiči. `override` je užitečná kontrola překlepů.

Při rozšiřování společného výpisu může potomek nejprve zavolat `Zaklad::vypis()` a potom vypsat svůj vlastní údaj.

Jednoduchá analýza potomka většinou znamená: připravit počítadlo nebo součet, projít kolekci cyklem, zkontrolovat podmínku a na konci výsledek vypsat. U průměru je nutné před dělením ověřit, že počet není nula.

### Malý příklad potomka

Hlavička ukazuje dědičnost, vlastní atribut a `override`:

```cpp
class Potomek : public Zaklad
{
private:
    double parametr;

public:
    Potomek(const std::string& nazev, double parametr);
    void vypis() const override;
    void analyzuj() const override;
};
```

Konstruktor pošle společný údaj rodiči a uloží vlastní údaj:

```cpp
Potomek::Potomek(const std::string& nazev, double parametr)
    : Zaklad(nazev), parametr(parametr)
{
}
```

Ukázka jednoduchého počítání hodnot splňujících podmínku:

```cpp
int pocet = 0;

for (double hodnota : historie)
{
    if (hodnota > 10)
    {
        pocet++;
    }
}
```

Číslo 10 je jen příklad. Ve skutečném programu sem patří podmínka popsaná v požadavcích.

---

## 8. Polymorfismus a paměť

Společná kolekce různých potomků má typ `vector<Zaklad*>`. Jednotlivé objekty se vytvoří pomocí `new Potomek(...)` a vloží přes `push_back`.

Při průchodu použij proměnnou typu `Zaklad*`. Přes šipku zavolej virtuální metody. Přestože je proměnná základního typu, díky `virtual` se spustí metoda skutečného potomka.

Pokud se potomci mažou přes ukazatel na základní třídu, základní destruktor musí být virtuální. Pro každý objekt vytvořený pomocí `new` musí později proběhnout `delete`.

Správný konec programu: projít původní vektor ukazatelů, na každém prvku zavolat `delete`, teprve potom případně použít `clear()` a nakonec zkontrolovat statický čítač. Samotné `clear()` odstraní ukazatele z vektoru, ale nesmaže dynamicky vytvořené objekty.

### Obecný polymorfní průchod

```cpp
std::vector<Zaklad*> objekty;

objekty.push_back(new PotomekA(/* parametry */));
objekty.push_back(new PotomekB(/* parametry */));

for (Zaklad* objekt : objekty)
{
    objekt->vypis();
    objekt->analyzuj();
}

for (Zaklad* objekt : objekty)
{
    delete objekt;
}

objekty.clear();
```

Komentář `/* parametry */` je schválně nevyplněný: při použití musíš doplnit hodnoty odpovídající konstruktoru konkrétního potomka.

---

## 9. Přetěžování operátorů

Operátor je metoda nebo spřátelená funkce, díky které lze s objektem použít běžný znak.

### Porovnání

Hlavička má obecný tvar `bool operator==(const Potomek& druhy) const;`. Metoda pouze porovná požadovaný atribut aktuálního objektu s atributem `druhy` a vrátí výsledek porovnání.

```cpp
bool Potomek::operator==(const Potomek& druhy) const
{
    return parametr == druhy.parametr;
}
```

### Operátor měnící objekt

Například `+=` nebo `*=` má často návratový typ `Potomek&`. Uvnitř provede požadovanou změnu a poslední krok je `return *this;`, tedy návrat právě upraveného objektu.

```cpp
Potomek& Potomek::operator+=(double hodnota)
{
    // zde proveď požadovanou změnu
    return *this;
}
```

### Výpis do proudu

V hlavičce se deklaruje funkce začínající `friend ostream& operator<<`. Přijímá proud a konstantní referenci na vypisovaný objekt. V `.cpp` už se nepíše `friend` ani předpona `Potomek::`. Funkce vkládá údaje do `os` a nakonec vrátí `os`.

```cpp
std::ostream& operator<<(std::ostream& os, const Potomek& objekt)
{
    os << "Nazev: " << objekt.nazev;
    return os;
}
```

Pokud je `nazev` nepřístupný, použij veřejný getter nebo vhodně umístěnou deklaraci `friend`.

Operátory je nejjednodušší testovat na obyčejných lokálních objektech bez `new`.

---

## 10. Jak vymyslet zpracování kolekce

Než začneš psát, odpověz si: Co funkce dostane? Má něco vrátit, nebo upravuje původní data? Jakou kolekci musí projít? Jak přesně pozná hledanou hodnotu?

### Počítání nebo součet

Připrav proměnnou s nulou, projdi hodnoty pomocí `for`, uvnitř ověř podmínku a podle potřeby zvyšuj počet nebo přičítej do součtu.

```cpp
int pocet = 0;
double soucet = 0;

for (double x : data)
{
    if (/* podmínka */)
    {
        pocet++;
        soucet += x;
    }
}

if (pocet > 0)
{
    double prumer = soucet / pocet;
}
```

### Nejdelší souvislá řada

Potřebuješ dvě počítadla: délku právě probíhající řady a nejlepší dosud nalezenou délku. Když prvek splňuje podmínku, první počítadlo zvýšíš a případně aktualizuješ maximum. Jakmile podmínku nesplní, aktuální délku vynuluješ. Nakonec vrátíš maximum.

```cpp
int aktualni = 0;
int maximum = 0;

for (double x : data)
{
    if (/* x splňuje podmínku */)
    {
        aktualni++;
        if (aktualni > maximum)
            maximum = aktualni;
    }
    else
    {
        aktualni = 0;
    }
}
```

### Mazání při průchodu

Vezmi si referenci na původní vektor. Iterátor začíná na `begin()` a pokračuje, dokud není `end()`. Když se má prvek smazat, nastav iterátor na výsledek `erase(it)`. Když se nemaže, proveď `++it`. Po `erase` už další posun nedělej, protože metoda sama vrací novou platnou pozici.

```cpp
for (auto it = data.begin(); it != data.end(); )
{
    if (/* prvek se má odstranit */)
        it = data.erase(it);
    else
        ++it;
}
```

### Filtrování do nového vektoru

Vytvoř prázdný výsledný vektor. Projdi původní kolekci a prvky splňující podmínku vlož přes `push_back`. Na konci vrať výsledný vektor. Tato varianta nemění původní data.

```cpp
std::vector<double> vysledek;

for (double x : data)
{
    if (/* x patří do výsledku */)
        vysledek.push_back(x);
}

return vysledek;
```

### Porovnávání sousedních hodnot

Použij indexový cyklus od hodnoty 1, protože porovnáváš pozice `i` a `i - 1`. Nejdřív ověř, že kolekce obsahuje alespoň dva prvky a že případným dělením nevznikne dělení nulou.

### Řazení a lambdy

Pro `sort`, `find_if` nebo `count_if` je potřeba `<algorithm>`. Lambda má tvar „hranaté závorky, parametry, složené závorky“. Vrací `true`, když prvek splňuje podmínku. U řazení `true` znamená, že první porovnávaný prvek má být před druhým.

```cpp
std::sort(data.begin(), data.end());

int pocet = std::count_if(
    data.begin(), data.end(),
    [](double x) { return x > 10; }
);
```

Když si lambdou nejsi jistá, je u jednoduchého počítání naprosto v pořádku použít obyčejný `for` cyklus. Důležitější je správná podmínka a funkční program.

---

## 11. Nejčastější chyby

- Středník mezi hlavičkou metody a jejím tělem v `.cpp`.
- Chybějící `const` v implementaci, přestože je v deklaraci.
- Jiný název metody v `.h`, `.cpp` a při volání.
- Zapomenuté `Zaklad::` před metodou implementovanou mimo třídu.
- Záměna tečky a šipky.
- Statický atribut deklarovaný jako funkce kvůli `()`.
- Připojení `.cpp` souboru místo `.h`.
- Chybějící `virtual` u destruktoru základní třídy.
- Použití objektu po `delete`.
- Mazání z vektoru v obyčejném range-based cyklu.
- Průměr počítaný bez kontroly nulového počtu.

---

## 12. Krátká kontrola před dokončením

- Program se přeloží a spustí.
- Každá třída má deklarace v `.h` a těla metod v `.cpp`.
- Základní třída má virtuální destruktor.
- Čistě virtuální metoda obsahuje `= 0`.
- Potomci dědí veřejně a přepsané metody mají `override`.
- Deklarace a implementace mají shodné parametry a `const`.
- Virtuální metody se zkoušejí přes ukazatel na základní třídu.
- Každému `new` odpovídá `delete`.
- Po úklidu je statický čítač zpět na nule.
