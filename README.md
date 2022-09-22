# AutoelektronikaProjekat

## Uvod
Ovaj projekat ima za zadatak da simulira rad brisača u automobilu. Kao okruženje koristi se VisualStudio2019. Zadatak ovog projekta osim same funkcionalnosti je bila i implementacija Misra standarda prilikom pisanja koda.	

### Ideja i zadaci:
1. Podaci o trenutnom stanju količine kiše se dobijaju svake 2 sekunde sa kanala 0 serijske komunikacije izraženi kao vrednost u opsegu od 0 do 1K.
2. Uzimati poslednjih 10 pristiglih vrednosti i računati prosek.
3. Zatim je neophodno izvršiti kalibraciju senzora kiše. Senzor kiše ima 3praga odnosno 4 stanja (nema kiše, slaba kiša, srednja kiša, jaka kiša).
Pragovi su pre kalibracije podešeni sledećim redom prag1=250, prag2=500, prag3=750. Da bi se podesila vrednost praga neophodno je uneti poruku 
na UniCom 1 u obliku NK zatim prag koji podešavamo (1,2,3), a zatim vrednost od 0 do 1000 i za kraj poruke potrbno je uneti CR.
4. Brisači mogu da rade Manuelno ili Automatski. U početku brisači su podešeni da rade automatski u zavisnosti od količine kiše. 
Da bismo ovo promenili neophodmo je uneti poruku u obliku MCR za manuelni režim, odnosno ACR ukoliko želimo da se vratimo u automatski režim rada.
5. Na UniCom1 periodom od 5000ms će se ispisivati trenutna količina kiše dobijena sa senzora, zatim režim rada brisača kao i dal i su oni uključeni.
U automatskom režimu oni su uključeni ukoliko je trenutna vrednost kiše veća od prvog praga, dok u manuelnom zavisi iključivo od potreba korisnika.
6. Ukoliko smo u manuelnom režimu rada korisnik upravlja radom brisača. Brisači imaju 3 brzine koje se uključuju prekidačima
(brzina1- donji prekidač prvog stupca, brzina2- donja 2 prekidača prvog stupca, brzina3- donja  3 prekidača prvog stupca). 
Ukoliko uključimo brisače svetleće dioda u drugom stupcu.
7. Ukoliko smo u automatskom režimu rada takodje imamo 3 brzine rada brisača koje su odredjene na osnovu odnosa trenutne količine kiše kao i
pragova koje smo prethodno podesili. Ukoliko je vrednos kiše manja od praga1, brisači neće raditi, ukoliko je izmedju praga1 i praga2 svetleće jedna dioda
u trećeme stupcu i predstavlja brzinu1, ukoliko je izmedju praga2 i praga3 svetleće dve diode u trećem stupcu i označava brzinu2,
a ukoliko je vrednost kiše veća od praga3 svetleće 3diode u trećem stupcu i označava brzinu3.
8. Na LCD displeju će pisati režim rada brisača(0 automatski, 1 manuelni), zatim brzina rada brisača kao i da li je brisac upaljen ili ugasen

#### Periferije
Periferije koje je potrebno koristiti su LED_bar, 7seg displej i AdvUniCom softver za simulaciju serijske komunikacije.
Prilikom pokretanja LED_bars_plus.exe navesti rRRr kao argument da bi se dobio led bar sa 2 izlazna i 2 ulazna stupca crvene boje. 7Seg displej je potrebno otvoriti sa 3 cifre. Što se tiče serijske komunikacije, potrebno je otvoriti i kanal 0 i kanal 1. Kanal 0 se automatski otvara pokretanjem AdvUniCom.exe,
a kanal 1 otvoriti dodavanjem broja jedan kao argument: AdvUniCom.exe 1

##### Kratak opis taskova:
1. void prvSerialReceiveTask_0(void* pvParameters)- prima 10 vrednosti sa UniCom0 i računa prosečnu količinu kiše.
2. void prvSerialReceiveTask_1(void* pvParameters)- prima poruku u obliku NK početak poruke, zatim (1,2,3) prag,
zatim vrednost koju želimo da postavimo za odredjeni prag i na kraju za kraj poruke se unosi CR. 
Takodje unosom MCR ili ACR podeđavamo režim rada brisača. 
3. void SerialSend_Task(void* pvParameters)- s obzirom da vrednost količine kiše treba da se dobije svakih 
200ms sa UniCom0, od strane FreeRTOS-a je to omogućeno tako što se u ovom tasku svakih 200ms šalje "senzor".
Kada se pokrene AdvUniCom.exe potrebno je označiti opciju AUTO, odnosno svaki put kad stigne "senzor",
imao informaciju da je poslata vrednost. Vrednost koja se pošalje je zapravo vrednost trenutne količine kiše.
Povremeno je potrebno ručno u AdvUniCom softveru menjati ovu vrednost kako bi se simulirala promena količine kiše.
4. void serialsend1_tsk(void* pvParameters)- ima za zadatak da na UniCom1 ispisuje trenutnu vrednost količine kiše,
kao i da li rade i u kom režimu i to radi konstantno periodom od 5000ms.
5. void set_led_Task(void* pvParameters)- task u kome se podešavaju odgovarajuče diode kao i LCD displej.
Na osnovu režima rada kao i vrednosti trenutne količine kiše i pragova senzora u slučaju automatskog režima radapali
odgovarajuće diode u drugom i trećem stupcu, dok u slučaju manuelnog režima rada pali brisače(diode)
pritiskanje tastera prvog stupca. Na LCD-u ispisuje 0(automaski režim), 1(manuelni režim), zatim brzinu rada
brisača kao i trenutnu vrednost količine kiše. 
6. void main_demo(void)- U ovoj funkciji se vrši inicijalizacija svih periferija koje se koriste,
kreiraju se taskovi, semafori i red, definiše se interrupt za serijsku komunikaciju i poziva vTaskStartScheduler()
funkcija koja aktivira planer za raspoređivanje taskova.

###### Predlog simulacije celog sistema
Prvo otvoriti sve periferije na način opisan gore. Pokrenuti program. U prozoru AdvUniCom softvera kanala 0, neophodno je uneti \00 početak poruke, zatim 10vrednosti u hex obliku npr.
\01\02\03\04\05\06\07\08\09\0a a zatim \ff koji označava kraj poruke. Vrednosti se mogu slati samo od 01-0a. Na taj način dobijamo trenutnu količinu kiše.
Zatim na UniCom1 neophodno je kalibrisati senzor. Da bi ove pragove promenili unosimo prouku u obliku NK1300CR, NK2600CR, NK3800CR.
Na ovaj način smo namestili pragove senzora. Takodje neophodno je uneti i režim rada brisača, na početku režim rada je automatski, režim se menja na UniCom1 porukom MCR odnosno ACR za povratak u automatski režim rada.
Na osnovu svega unetog do sad na UniCom1 se moze iscitati trenutna vrednost kiše, režim rada brisača kao i da li oni uopšte i rade. Zatim u zavisnosti da li je uključen automatski ili manuelni režim rada imamo kontrolu brzine brisača.
U manuelnom režimu rada brzina se menja uključivanjem prekidača na prvom stupcu desnim klikom na taster. Donji taster je prva brzina, donja dva tastera druga brzina a donja tri tastera treća.
U drugom stupcu ukoliko su upaljeni brisači (brzina rada 1 ili veća) svetleće dioda. Ukoliko je režim automatski u zavisnosti od pragova i trenutne količine kiše imamo takodje tri brzine rada. Ukoliko je trenutna vrednost kiše manja od prbog praga neće raditi brisači,
Na LCD-u će biti ispisano 0 ako je režim rada automatski, zatim 1 ako je manuelni zatim ide trenutna brzina rada brisača i na kraju 1 ako brisac radi ili 0 ako ne radi.
