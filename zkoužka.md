# C++ OOP – praktický průvodce tvorbou programu

Tyto poznámky shrnují obecný postup při tvorbě menšího objektově orientovaného programu. Neuč se názvy tříd nazpaměť. Uč se poznat, co jednotlivé požadavky znamenají a do kterého souboru jejich řešení patří.

---

## 0. Doporučené pořadí práce

Postupuj v tomto pořadí a po každém kroku dej **Build → Build Solution (`Ctrl+Shift+B`)**:

1. Založ projekt a soubory.
2. Napiš základní třídu (`.h` + `.cpp`).
3. Napiš prvního potomka a hned ho otestuj lokálně.
4. Napiš druhého potomka a hned ho otestuj lokálně.
5. Teprve potom vytvoř `vector<Zaklad*>`, použij `new`, polymorfní cyklus a `delete`.
6. Přidej operátory.
7. Přidej jednoduché zpracování dat.
8. Mazání, filtrování a řazení nech úplně nakonec.

**Důležitější než množství rozepsaných funkcí je program, který se přeloží a spustí.** Po každé dokončené části projekt znovu sestav a oprav chyby dříve, než pokračuješ.

---

## 1. Jak rozložit popis úlohy

Nejdřív si v textu barevně nebo na papír označ:

- **podstatná jména** → většinou třídy a atributy,
- **slovesa** → většinou metody,
- **„virtuální / čistě virtuální“** → dědičnost a `virtual`,
- **„statický čítač“** → `static int`, konstruktor `++`, destruktor `--`,
- **„historie / seznam hodnot“** → `std::vector<...>`,
- **„vypočítá a vrátí“** → návratový typ není `void`,
- **„vypíše“ nebo „upraví“** → často `void`,
- **„nemění objekt“** → `const` na konci metody,
- **„pro všechny potomky“** → parametr nebo vektor základního typu.

### Příklad převodu věty na kód

> Třída uchovává chráněný textový identifikátor a historii desetinných hodnot.

Odvození:

```cpp
protected:
    std::string identifikator;
    std::vector<double> historie;
```

> Každý potomek musí mít vlastní neměnící metodu pro analýzu.

Odvození:

```cpp
virtual void analyzuj() const = 0;
```

> Metoda vypočítá a vrátí desetinný výsledek.

Odvození: výsledek je desetinné číslo, takže ne `void`:

```cpp
virtual double vypocitejHodnotu() const = 0;
```

---

## 2. Soubory ve Visual Studiu

Pro každou třídu bezpečně používej dvojici:

```text
Zaklad.h       Zaklad.cpp
PotomekA.h     PotomekA.cpp
PotomekB.h     PotomekB.cpp
main.cpp
```

Do `main.cpp` připojuj pouze hlavičky:

```cpp
#include "PotomekA.h"
#include "PotomekB.h"
```

**Nikdy nepřipojuj `.cpp`:**

```cpp
#include "PotomekA.cpp" // špatně
```

Visual Studio přeloží všechny `.cpp` soubory přidané do projektu automaticky.

### Co patří do `.h`

- atributy,
- deklarace konstruktoru a destruktoru,
- názvy metod, parametry a návratové typy,
- `virtual`, `override`, `= 0`, `const`, `static`, `friend`.

```cpp
#pragma once

class Trida
{
private:
    // atributy

public:
    // deklarace metod zakončené ;
};
```

### Co patří do `.cpp`

- skutečná logika metod,
- před názvem metody je `Trida::`,
- za hlavičkou implementované metody není středník.

```cpp
#include "Trida.h"

void Trida::metoda()
{
    // logika
}
```

Častá chyba:

```cpp
void Trida::metoda(); // špatně: středník ukončil deklaraci
{
}
```

---

## 3. `private`, `protected`, `public`

| Přístup | Kdo údaj vidí |
|---|---|
| `private` | pouze daná třída |
| `protected` | daná třída a její potomci |
| `public` | také `main.cpp` a ostatní kód |

Použití:

- společná data, se kterými pracují potomci → `protected`,
- vlastní atribut potomka → většinou `private`,
- metody volané z `main()` → `public`.

---

## 4. Základní abstraktní třída – jak ji odvodit

Když program potřebuje základní třídu, obvykle do ní patří:

1. společné atributy,
2. konstruktor,
3. virtuální destruktor,
4. gettery a společné metody,
5. virtuální nebo čistě virtuální metody.

### Univerzální kostra hlavičky

Názvy v hranatých závorkách nahraď podle konkrétního programu:

```cpp
#pragma once
#include <string>
#include <vector>

class [Zaklad]
{
protected:
    std::string [nazev];
    std::vector<double> [historie];
    static int [citac];

public:
    [Zaklad](const std::string& nazev);
    virtual ~[Zaklad]();

    static int getPocetAktivnich();
    std::vector<double>& getHistorie();

    virtual void vypisInfo() const;
    virtual void analyzuj() const = 0;
};
```

`= 0` znamená, že metoda je čistě virtuální a základní třída je abstraktní. Nelze vytvořit:

```cpp
Zaklad objekt; // nejde
```

Lze ale vytvořit ukazatel na potomka:

```cpp
Zaklad* objekt = new Potomek(...);
```

### Statický čítač v `.cpp`

V `.h` je pouze deklarace:

```cpp
static int pocetAktivnich;
```

V jednom `.cpp` musí být definice bez závorek:

```cpp
int Zaklad::pocetAktivnich = 0;
```

Pozor:

```cpp
static int pocetAktivnich(); // špatně: toto je funkce
```

Konstruktor čítač zvýší, destruktor sníží:

```cpp
Zaklad::Zaklad(const std::string& noveJmeno)
    : nazev(noveJmeno)
{
    pocetAktivnich++;
}

Zaklad::~Zaklad()
{
    pocetAktivnich--;
}
```

### Přidání jedné a více hodnot

Jedna hodnota:

```cpp
void Zaklad::pridejHodnotu(double hodnota)
{
    historie.push_back(hodnota);
}
```

Více hodnot:

```cpp
void Zaklad::pridejHodnoty(const std::vector<double>& hodnoty)
{
    for (double hodnota : hodnoty)
    {
        pridejHodnotu(hodnota);
    }
}
```

### Getter historie

Pokud algoritmus musí data měnit nebo mazat:

```cpp
std::vector<double>& getHistorie();
```

Pokud je má pouze číst, bezpečnější je:

```cpp
const std::vector<double>& getHistorie() const;
```

Pro jednodušší řešení lze použít první variantu i pro čtení.

---

## 5. `const` bez zmatku

### `const` u parametru

```cpp
void metoda(const std::string& text);
```

Znamená: metoda dostane text odkazem, ale nesmí ho změnit.

### `const` na konci metody

```cpp
void vypisInfo() const;
```

Znamená: metoda nesmí měnit atributy aktuálního objektu.

Používej u:

- getterů určených pouze ke čtení,
- výpisů,
- analýz, které jen počítají,
- porovnání `operator==`.

Nepoužívej na konci u metody, která přidává, maže nebo mění data.

**Co je v `.h`, musí přesně souhlasit s `.cpp`:**

```cpp
// .h
void vypisInfo() const;

// .cpp
void Trida::vypisInfo() const
{
}
```

---

## 6. Odvozená třída

V popisu programu najdi:

- jméno potomka,
- jeho vlastní atribut,
- které virtuální metody má přepsat.

### Hlavička potomka

```cpp
#pragma once
#include "Zaklad.h"

class Potomek : public Zaklad
{
private:
    double vlastniAtribut;

public:
    Potomek(const std::string& nazev, double hodnota);

    void vypisInfo() const override;
    void analyzuj() const override;
};
```

### Konstruktor potomka

Část dat pošli rodiči, vlastní atribut nastav potomkovi:

```cpp
Potomek::Potomek(const std::string& noveJmeno, double hodnota)
    : Zaklad(noveJmeno),
      vlastniAtribut(hodnota)
{
}
```

### Rozšířený výpis

Nejdřív zavolej společnou část rodiče, potom přidej vlastní údaje:

```cpp
void Potomek::vypisInfo() const
{
    Zaklad::vypisInfo();
    std::cout << " | Vlastni udaj: " << vlastniAtribut << std::endl;
}
```

### Analýza přes jednoduchý cyklus

Většina analýz má podobu:

```cpp
void Potomek::analyzuj() const
{
    int pocet = 0;

    for (double hodnota : historie)
    {
        if (/* požadovaná podmínka */)
        {
            pocet++;
        }
    }

    std::cout << pocet << std::endl;
}
```

Obecné příklady podmínek:

```cpp
hodnota < mez
hodnota > mez
```

Průměr potřebuje součet i počet a ochranu před dělením nulou:

```cpp
double soucet = 0;
int pocet = 0;

// v cyklu přičítej a zvyšuj počet

if (pocet > 0)
{
    double prumer = soucet / pocet;
}
```

---

## 7. První průběžná kontrola

Než začneš s ukazateli a operátory, vytvoř potomky lokálně v bloku:

```cpp
std::cout << Zaklad::getPocetAktivnich() << std::endl; // 0

{
    PotomekA a("A", 10);
    PotomekB b("B", 20);

    a.pridejHodnoty({100, -20, 300});
    b.pridejHodnoty({500, 600});

    a.vypisInfo();
    a.analyzuj();
    b.vypisInfo();
    b.analyzuj();

    std::cout << Zaklad::getPocetAktivnich() << std::endl; // 2
}

std::cout << Zaklad::getPocetAktivnich() << std::endl; // 0
```

Pokud funguje `0 → 2 → 0`, fungují konstruktory, destruktor i čítač.

---

## 8. Polymorfismus, `new` a `delete`

Zadání typu „uložte různé potomky do společného vektoru“ znamená:

```cpp
std::vector<Zaklad*> objekty;

objekty.push_back(new PotomekA("A", 10));
objekty.push_back(new PotomekA("B", 20));
objekty.push_back(new PotomekB("C", 30));
```

Tečka je pro objekt, šipka pro ukazatel:

```cpp
objekt.metoda();
ukazatel->metoda();
```

Polymorfní průchod:

```cpp
for (Zaklad* objekt : objekty)
{
    objekt->vypisInfo();
    objekt->analyzuj();
}
```

Díky `virtual` se pro každý objekt zavolá správná verze potomka.

### Povinný úklid

Každý `new` musí mít `delete`:

```cpp
for (Zaklad* objekt : objekty)
{
    delete objekt;
}

objekty.clear();
```

Pořadí je důležité: nejdřív `delete`, potom `clear`. Samotné `clear()` objekty nesmaže.

---

## 9. Operátory – pouze ty, které jsou požadované

Nevymýšlej `[]` nebo `()`, pokud nejsou požadované. Často se používá porovnání, změnový operátor a výpis.

### Porovnání `==`

V `.h`:

```cpp
bool operator==(const Potomek& druhy) const;
```

V `.cpp`:

```cpp
bool Potomek::operator==(const Potomek& druhy) const
{
    return vlastniAtribut == druhy.vlastniAtribut;
}
```

Změň pouze porovnávaný údaj podle významu konkrétní třídy.

### Změna objektu `+=` nebo `*=`

V `.h`:

```cpp
Potomek& operator+=(double hodnota);
```

V `.cpp`:

```cpp
Potomek& Potomek::operator+=(double hodnota)
{
    // uprav objekt nebo zavolej již hotovou metodu
    return *this;
}
```

Pro `*=` je struktura stejná, změní se pouze znak a logika.

### Výpis `<<`

V `.h` připoj `<ostream>` a deklaruj:

```cpp
friend std::ostream& operator<<(std::ostream& os, const Potomek& objekt);
```

V `.cpp` se nepíše `Potomek::` ani `friend`:

```cpp
std::ostream& operator<<(std::ostream& os, const Potomek& objekt)
{
    os << "Potomek[" << objekt.nazev << "]";
    return os;
}
```

Operátory testuj na lokálních objektech bez `new`, ideálně ve vlastním `{ ... }` bloku.

---

## 10. Jak sestavit algoritmus bez hotového výsledku

Nejdřív si napiš česky čtyři otázky:

1. Co funkce dostane?
2. Co vrátí, nebo pouze něco změní?
3. Co musí projít cyklem?
4. Jaká je přesná podmínka uvedená v popisu?

### Typ A: Nejdelší souvislá řada

Potřebuješ dvě proměnné:

- aktuální délku,
- nejlepší nalezenou délku.

```cpp
int nejdelsiRada(Zaklad& objekt)
{
    int aktualni = 0;
    int maximum = 0;

    for (double hodnota : objekt.getHistorie())
    {
        if (/* hodnota splňuje podmínku */)
        {
            aktualni++;

            if (aktualni > maximum)
            {
                maximum = aktualni;
            }
        }
        else
        {
            aktualni = 0;
        }
    }

    return maximum;
}
```

### Typ B: Smazání prvků splňujících podmínku

Jednoduchá varianta přes iterátor:

```cpp
void odstranHodnoty(Zaklad& objekt)
{
    std::vector<double>& data = objekt.getHistorie();

    for (auto it = data.begin(); it != data.end(); )
    {
        if (/* podmínka pro smazání */)
        {
            it = data.erase(it);
        }
        else
        {
            ++it;
        }
    }
}
```

Po `erase` neposouvej iterátor ručně; `erase` už vrátí další platnou pozici.

### Typ C: Filtrování do nového vektoru

Použij, když původní data nemáš mazat:

```cpp
std::vector<Zaklad*> vyberObjekty(const std::vector<Zaklad*>& objekty)
{
    std::vector<Zaklad*> vysledek;

    for (Zaklad* objekt : objekty)
    {
        if (/* podmínka pro zařazení */)
        {
            vysledek.push_back(objekt);
        }
    }

    return vysledek;
}
```

### Typ D: Největší změna mezi sousedními hodnotami

Potřebuje indexy, protože porovnává `i` a `i-1`:

```cpp
double nejvetsiZmena(Zaklad& objekt)
{
    std::vector<double>& data = objekt.getHistorie();

    if (data.size() < 2)
    {
        return 1.0;
    }

    double maximum = 1.0;

    for (size_t i = 1; i < data.size(); i++)
    {
        double pomer = data[i] / data[i - 1];

        if (pomer > maximum)
        {
            maximum = pomer;
        }
    }

    return maximum;
}
```

---

## 11. Co je obecné a co se mění

U podobných programů zůstává stejná kostra:

- rozdělení tříd do `.h` a `.cpp`,
- dědičnost a `virtual`/`override`,
- statický čítač objektů,
- společný vektor ukazatelů,
- polymorfní průchod,
- uvolnění paměti přes `delete`.

Mění se hlavně názvy tříd, jejich vlastní atributy, podmínky uvnitř cyklů a požadované operátory. Proto nejdřív sestav obecnou kostru a teprve potom doplň výpočty odpovídající významu programu.

---

## 12. Nejčastější chyby

### Proměnná omylem napsaná jako funkce

```cpp
static int citac(); // špatně
static int citac;   // správně
```

### Neshoda `.h` a `.cpp`

```cpp
// .h
void vypis() const;

// .cpp – musí mít také const
void Trida::vypis() const
```

### Středník před tělem metody

```cpp
void Trida::metoda(); // špatně v .cpp
{
}
```

### Záměna podobných názvů

```cpp
pridejHodnotu(double)                 // jedna hodnota
pridejHodnoty(vector<double>)         // více hodnot
```

### Tečka versus šipka

```cpp
objekt.metoda();
ukazatel->metoda();
```

### `clear()` není `delete`

```cpp
for (Zaklad* objekt : objekty)
{
    delete objekt;
}
objekty.clear();
```

### Volání algoritmu po `delete`

Algoritmy a výpisy musí proběhnout před mazacím cyklem. Po `delete` už objekt neexistuje.

---

## 13. Kontrolní seznam před odevzdáním

- [ ] Projekt se přeloží přes `Ctrl+Shift+B`.
- [ ] Program se spustí přes `Ctrl+F5` a nespadne.
- [ ] Každá třída má správný `.h` a `.cpp`.
- [ ] Základní třída má virtuální destruktor.
- [ ] Čistě virtuální metoda obsahuje `= 0`.
- [ ] Potomci používají `: public Zaklad` a `override`.
- [ ] Statický čítač ukazuje na začátku `0`.
- [ ] Polymorfní vektor má typ `vector<Zaklad*>`.
- [ ] Pro každý `new` proběhne `delete`.
- [ ] Konečný čítač je znovu `0`.
- [ ] Posledních 10–15 minut už nepřidávám nové části, pouze testuji.
