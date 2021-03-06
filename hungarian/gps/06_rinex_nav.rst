RINEX navigációs fájl átalakítása, és egy adott műhold, adott időszakra vonatkozó adatainak leválogatása
========================================================================================================
*szerző: Takács Bence (takacs.bence@epito.bme.hu), BME Általános- és Felsőgeodézia Tanszék.*

Felmerült a feladat, hogy egy 2.1 verziójú (amerikai) GPS-műholdak adatait tartalmazó RINEX navigációs állományból keressük ki egy adott műhold, adott időpontra vonatkozó navigációs adatait! Ebben a segédanyagban lépésről-lépésre bemutatjuk hogyan oldható meg a feladat octave programban. A segédletet teljesen kezdőknek is ajánljuk.

**1.** Először írjunk egy szkriptet, ami sorról sorra végiolvassa a rinex formátumú navigációs állományunkat, bár megjegyezzük, hogy ez az alap szkript bármilyen text fájl beolvasására alkalmazható::

	fin = fopen ("brdc1520.15n", "r");
	while (! feof (fin) )
		text_line = fgetl (fin);
	endwhile
	fclose (fin);

Az első sorban olvasásra nyitjuk meg a brdc1520.15n állományunkat, a fájlt az 5. sorban zárjuk le. Megnyitán után a while ciklusban amíg a fájl vége jelet el nem érjük, a fgetl parancs segítségével soronként beolvassuk az állomány tartalmát, az aktuális sor tartalmát a text_line változóba tesszük. Egyelőre a beolvasott adatokkal semmit nem kezdünk.

Ha szeretnéd kiíratni az egyes sorok tartalmát a lépernyőre, a text_line = fgetl (fid); sorok végéről töröld ki a ;-t! 

**2.** Ezután alakítsuk át úgy a szkriptünket, hogy csak a fejléc ("header") utolsó sorának tartalmát írja ki! Ehhez nézzük meg, hogy az aktuális sor tartalmazza-e az "END OF HEADER" szavakat, ha igen, akkor írja ki az adott sor tartalmát. Ehhez a strfind beépített függvényt használjuk::

	fin = fopen ("brdc1520.15n", "r");
	while (! feof (fin) )
		text_line = fgetl (fin);
		if strfind(text_line, "END OF HEADER") 
			disp(text_line);
		endif
	endwhile
	fclose (fin);
	
**3.** Akkor most bevezetünk egy d változót, aminek értéke a program indításakor 0, ha elértük a fejléc végét, akkor értéke 1-re vált::

	fin = fopen ("brdc1520.15n", "r");
	d=0;
	while (! feof (fid) )
		text_line = fgetl (fin);

		if strfind(text_line, "END OF HEADER")
			d=1;
		endif

	endwhile
	fclose (fin);
	
**4.** Most tegyük hozzá, hogy a navigációs adatokat írja is ki egy állományba ("data.txt"). Ehhez az állományt írásra kell megnyitnunk, és persze a végén illik lezárni. Fájlba íráshoz az fprintf parancsot használjuk::

	fin = fopen ("brdc1520.15n", "r");
	fou = fopen ("data.txt", "w");
	d=0;
	while (! feof (fin) )
		text_line = fgetl (fin);

		if (d==1)
		    fprintf (fou, "%s\n", text_line);
		endif

		if strfind(text_line, "END OF HEADER")
			d=1;
		endif

	endwhile
	fclose (fin);
	fclose (fou);

**5.** Most alakítsuk át a programunkat olyanra, hogy minden navigációs adatot egy sorban írjon ki! Ehhez csak az fprintf parancssorból a sortörés (\n) kiírását kell eltávolítani::

	fin = fopen ("brdc1520.15n", "r");
	fou = fopen ("data.txt", "w");
	d=0;
	while (! feof (fin) )
		text_line = fgetl (fin);

		if (d==1)
		    fprintf (fou, "%s", text_line);
		endif

		if strfind(text_line, "END OF HEADER")
			d=1;
		endif

	endwhile
	fclose (fin);
	fclose (fou);

**6.** Most alakítsuk át a programunkat olyanra, hogy egy navigációs adatcsomagot egy sorban írjon ki! Ehhez meg kell néznünk, hog mikor jön egy újabb adatcsomag. Kiindulhatunk abból, hogy egy adatcsomag első sorában jóval több a szóközök száma, mint a többi sorban. Programunkban a szóközök számát az n változó tartalmazza. Ha ez 9 feletti, akkor az output fájlba be kell tennünk egy sortörést::

	fin = fopen ("brdc1520.15n", "r");
	fou = fopen ("data.txt", "w");
	d=0;
	while (! feof (fin) )
		text_line = fgetl (fin);
		n=length(strfind(text_line,' '));

		if (d==1)
		    if (n>9)
			fprintf(fou, "\n");
		    endif
		    fprintf (fou, "%s", text_line);
		endif

		if strfind(text_line, "END OF HEADER")
			d=1;
		endif

	endwhile
	fclose (fin);
	fclose (fou);

Annyi szépséghibája van az eredmény állománynak, hogy egy üres sorral kezdődik.

Egy adott műhold, adott időszakra vonatkozó navigációs adatai most már könyebben kereshetők az output fájlban. 

**7.** Most úgy írjuk meg a programot, hogy a fejléc vége után 8 sort olvasunk be, hiszen egy műhold egy kétórás időszakra vonatkozó adatai, nevezzük adatcsomagnak 8 sorból állnak a rinex fájlban::

	fin = fopen ("brdc1520.15n", "r");
	fou = fopen ("data.txt", "w");
	d=0;
	while (! feof (fin) )
		text_line = fgetl (fin);

		if (d==1)
		    fprintf (fou, "%s", text_line);

		    for i=1:7
			text_line = fgetl (fin);
			fprintf(fou, "%s", text_line);
		    endfor
		    fprintf(fou, "\n");
		endif

		if strfind(text_line, "END OF HEADER")
			d=1;
		endif

	endwhile
	fclose (fin);
	fclose (fou);

**8.** Most alakítsuk át a programunkat úgy, hogy csak egy adott műhold adatait írjuk ki! Ehhez az első sor első adatát be kell olvassuk egy változóba (esetünkben *prn*) és a kiírásnál csak akkor írjuk ki az adatokat, ha a *prn* változó értéke megegyezik a program elején definiált *prn0* értékkel. A `rinex <https://igscb.jpl.nasa.gov/igscb/data/format/rinex210.txt>`_ fájlban fix hosszúságú egy adat értéke, a műholdak azonosítója két karakter hosszú, egész szám. A text_line(1:2) parancs az adott sor első két karakterét veszi ki, az eredmény maga is string lesz. Ezért használjuk a *str2num* függvényt, ami a stringet számmá alakítja::

	fin = fopen ("brdc1520.15n", "r");
	fou = fopen ("data.txt", "w");
	d=0;
	prn0=8;
	while (! feof (fin) )
		text_line = fgetl (fin);

		if (d==1)
		    prn=str2num(text_line(1:2));
		    if prn==prn0
			fprintf (fou, "%s", text_line);
		    endif
		    for i=1:7
			text_line = fgetl (fin);
			if prn==prn0
			    fprintf(fou, "%s", text_line);
			endif
		    endfor
		    if prn==prn0
			fprintf(fou, "\n");
		    endif
		endif

		if strfind(text_line, "END OF HEADER")
			d=1;
		endif

	endwhile
	fclose (fin);
	fclose (fou);

**9.** A következő lépésben úgy alakítsuk át a programot, hogy az adatcsomag referencia idejét is nézi és csak egy adott időponthoz tartozó adacsomagot írjuk ki. Egy adatcsomag két óráig használható fel, olyan adatcsomagot keressünk, aminek referencia időpontja az adott időpontot megelőzi, és a két időpont különbsége kevesebb, mint 2 óra. A referencia időpont (*TOE = Time of Ephemeris*) a 4. sor első adata, a rinex navigációs fájlban az első kivételével minden adatsor 3 szóközzel kezdődik, utána 19 karakter hosszúságú egy szám.

**10.** Most pedig arra lenne szükségünk, hogy a navigációs adatokat tároljuk pl. egy tömbben vagy más alkalmas változókban, hogy a későbbiekben számolni tudjunk az adatokkal.


