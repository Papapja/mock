// ============================================================
//                 TAHÁK – MAIN.CPP
// ============================================================


// ==================== ZÁKLAD MAINU ===========================

#include <iostream>
#include "Trida.h"          // změnit podle zadání
#include "DalsiTrida.h"     // pokud je potřeba

using namespace std;

int main()
{

    // ========================================================
    // 1. VYTVOŘENÍ OBJEKTU
    // ========================================================

    // Obecně:
    // Trida nazevObjektu(parametry);

    // Příklady:
    // BeznyUcet ucet1("123456", 50);
    // SporiciUcet ucet2("987654", 3.5);

    // Parametry zjistím podle konstruktoru v .h:
    //
    // BeznyUcet(const string& cislo, double poplatek);
    //
    // ↓
    //
    // BeznyUcet ucet1("123456", 50);



    // ========================================================
    // 2. VOLÁNÍ FUNKCE / METODY
    // ========================================================

    // Bez parametru:
    //
    // objekt.funkce();

    // Příklad:
    // ucet1.vypisInfo();
    // ucet1.analyzujUcet();



    // ========================================================
    // 3. FUNKCE S PARAMETREM
    // ========================================================

    // objekt.funkce(hodnota);

    // Příklad:
    // ucet1.pridejTransakci(500);
    // ucet1.pridejTransakci(-200);



    // ========================================================
    // 4. VÍCE TRANSAKCÍ
    // ========================================================

    // ucet1.pridejTransakci(1000);
    // ucet1.pridejTransakci(-500);
    // ucet1.pridejTransakci(300);
    // ucet1.pridejTransakci(-100);



    // ========================================================
    // 5. COUT – VÝPIS
    // ========================================================

    // Text:
    // cout << "Ahoj" << endl;

    // Proměnná:
    // cout << promenna << endl;

    // Text + proměnná:
    // cout << "Hodnota: " << promenna << endl;



    // ========================================================
    // 6. OPERATOR <<
    // ========================================================

    // Pokud je ve třídě vytvořen:
    //
    // ostream& operator<<(ostream& vystup, const Trida& objekt);
    //
    // můžu napsat:

    // cout << objekt << endl;

    // Příklad:
    // cout << ucet1 << endl;



    // ========================================================
    // 7. OPERATOR +=
    // ========================================================

    // Pokud je v .h:
    //
    // Trida& operator+=(double hodnota);
    //
    // můžu napsat:

    // objekt += hodnota;

    // Příklad:
    // ucet1 += 500;



    // ========================================================
    // 8. OPERATOR ==
    // ========================================================

    // Pokud je v .h:
    //
    // bool operator==(const Trida& druhy) const;
    //
    // můžu:

    /*
    if (objekt1 == objekt2)
    {
        cout << "ANO" << endl;
    }
    else
    {
        cout << "NE" << endl;
    }
    */



    // ========================================================
    // 9. IF
    // ========================================================

    /*
    if (podminka)
    {
        // udělá se pokud platí
    }
    else
    {
        // udělá se pokud neplatí
    }
    */

    // Příklady podmínek:
    //
    // if (x > 0)
    // if (x < 0)
    // if (x == 10)
    // if (x != 10)
    // if (objekt1 == objekt2)



    // ========================================================
    // 10. FOR – KLASICKÝ
    // ========================================================

    /*
    for (int i = 0; i < 5; i++)
    {
        cout << i << endl;
    }
    */



    // ========================================================
    // 11. VECTOR
    // ========================================================

    // Pokud potřebuji vector:

    // vector<double> hodnoty = {100, 200, -50};

    // A metoda přijímá vector:
    //
    // objekt.pridejTransakce(hodnoty);



    // ========================================================
    // 12. FOR PRO VECTOR
    // ========================================================

    /*
    for (double hodnota : hodnoty)
    {
        cout << hodnota << endl;
    }
    */



    // ========================================================
    // 13. UKÁZKOVÝ MAIN PRO ÚČTY
    // ========================================================

    /*
    BeznyUcet ucet1("123456", 50);
    BeznyUcet ucet2("654321", 50);

    SporiciUcet sporeni("999999", 3.5);


    // přidání transakcí
    ucet1.pridejTransakci(1000);
    ucet1.pridejTransakci(-200);
    ucet1.pridejTransakci(500);


    // výpis informací
    ucet1.vypisInfo();


    // analýza
    ucet1.analyzujUcet();


    // +=
    ucet1 += 300;


    // <<
    cout << ucet1 << endl;


    // ==
    if (ucet1 == ucet2)
    {
        cout << "Ucty jsou stejne." << endl;
    }
    else
    {
        cout << "Ucty nejsou stejne." << endl;
    }


    // spořicí účet
    sporeni.pridejTransakci(5000);
    sporeni.pridejTransakci(2000);

    sporeni.vypisInfo();
    sporeni.analyzujUcet();
    */



    // ========================================================
    //          JAK POZNÁM CO PSÁT DO MAINU?
    // ========================================================

    /*
        PODÍVÁM SE DO .h NA:

                        public:

        To, co je v PUBLIC, můžu většinou používat v mainu.


        PŘÍKLAD:

        public:

            BeznyUcet(const string& cislo, double poplatek);

            void pridejTransakci(double hodnota);

            void analyzujUcet() const;

            void vypisInfo() const;


        PŘEKLAD DO MAINU:


        BeznyUcet(...)
              ↓
        vytvořím objekt:

        BeznyUcet ucet("123", 50);



        void pridejTransakci(double hodnota)
              ↓
        zavolám:

        ucet.pridejTransakci(500);



        void analyzujUcet() const
              ↓

        ucet.analyzujUcet();



        void vypisInfo() const
              ↓

        ucet.vypisInfo();

    */



    // ========================================================
    //               NEJDŮLEŽITĚJŠÍ PRAVIDLO
    // ========================================================

    /*
        .h
        = CO TŘÍDA UMÍ
        = proměnné + deklarace funkcí


        .cpp
        = JAK TO TŘÍDA DĚLÁ
        = těla funkcí


        main.cpp
        = POUŽÍVÁM HOTOVÉ TŘÍDY


        V MAINU:

        1. vytvořím objekty

        2. přidám data

        3. zavolám funkce

        4. případně použiju operátory

        5. vypíšu výsledky


        ZÁKLADNÍ VZOR:

        Trida objekt(parametry);

        objekt.funkce();

        objekt.funkce(hodnota);

        cout << objekt << endl;

        objekt += hodnota;

        if (objekt1 == objekt2)
        {
            ...
        }
    */


    return 0;
}
