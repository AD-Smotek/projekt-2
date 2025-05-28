hraní Šach s kamarádem na webovce

Naprogramoval jsem jednoduchou webovou šachovou hru pomocí HTML, CSS a JavaScriptu. Po načtení stránky se mi vykreslí šachovnice, která je složená z 8×8 čtverců. Každý čtverec (nebo buňka) je buď bílý, nebo černý podle pozice, a pokud na něm stojí figurka, zobrazí se pomocí šachového Unicode symbolu.
Ve hře mám definované výchozí postavení figurek v poli initialBoard, které je klasicky nastavené jako na začátku šachové partie. Každý prvek v tom poli je písmeno, které označuje typ figurky – třeba 'P' je bílý pěšec, 'k' je černý král.
Funkce initBoard() se postará o to, že se všechno správně načte – překopíruje výchozí pole na herní pole board, nastaví stav hry (např. že ještě neskončila) a zavolá renderBoard(), která zobrazí šachovnici na stránce.
Když kliknu na nějakou buňku, spustí se funkce handleCellClick(). Pokud už mám vybranou figurku, zkontroluje se, jestli se na danou cílovou pozici může pohnout podle pravidel – a pokud ano, provede se tah. Jinak si jen vyberu novou figurku, kterou chci táhnout.
Každý typ figurky má vlastní funkci, která ověřuje, jestli je daný tah povolený – třeba isValidPawnMove() pro pěšce, isValidKnightMove() pro koně atd. Tyto funkce kontrolují, jestli je cesta volná, jakým směrem figurka smí jít, nebo jestli může sebrat soupeřovu figurku.
Pokud pěšec dojde na poslední řadu, zobrazí se mi nabídka pro proměnu – můžu si vybrat, jestli ho chci změnit na dámu, věž, střelce nebo jezdce. Tah se dokončí až po tom výběru.
Na konci každého tahu kontroluju, jestli ještě zůstávají oba králové na šachovnici. Pokud jeden zmizí, hra končí a zobrazí se mi tlačítka – buď si můžu zahrát znovu, nebo hru ukončit.

link:https://youtu.be/lB2HuOETEGs





