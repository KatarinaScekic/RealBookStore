**Stručni kurs Razvoj bezbednog softvera**

**Izveštaj**

**Pronađene ranjivosti u projektu “RealBookStore”**

**Katarina Šćekić**





**Uvod:**

Ovaj izveštaj se bavi ranjivostima pronađenim u dole opisanoj veb aplikaciji.

O veb aplikaciji

RealBookStore je veb aplikacija koja pruža mogućnosti pretrage, ocenjivanja i komentarisanja knjiga.

Aplikacija RealBookStore omogućava sledeće:

`    `• Pregled i pretragu knjiga.

`    `• Dodavanje nove knjige.

`    `• Detaljan pregleda knjige kao i komentarisanje i ocenjivanje knjige.

`    `• Pregled korisnika aplikacije.

`    `• Detaljan pregled podataka korisnika.


**SQLI:**

**Primer napada:**

Za testiranje ranjivosti aplikacije na SQL Injection napade koristila sam prostu SQL komantu koja insertuje novu osobu u postojecu tabelu. Ranjivost sam pronasla u textarea delu dodavanja komentara o knjizi.\
Ova naredba cancel-uje ocekivani String u lose obezbedjenom kodu za obradu upita, zatim izvrsava moj SQL upit, i na kraju kometarima gasi ostatak koda u istoj naredbi:\
\
‘komentar); insert into persons (firstName, lastName, email) values (‘Thor’, ‘Odinson’, ‘<mjolnir@gmail.com>’) - -

(videti slike report/SQLI\_insert i report/SQLI\_it\_worked)

**Odbrana od SQLI napada:**

Jedan od vidova odbrane je koriscenje PrepareStatementa umesto standarnih Statementa. Kada im se setuju sva polja, poziva se funkcija za executeUpdate()\
Takodje, koriste se placeholderi umesto vrednosti pri insertovanju, kako bi se one izracunale tek kada se zavrsi SQL upit, na primer: \
\
String query = "insert into comments(bookId, userId, comment) values (?, ?, ?)";


**XSS:**

**Primer napada:**\
Ranjivost na XSS napade otkrila sam u delu za editovanje licnog profila korisnika. Ukoliko se npr. atribut email popuni dobro osmisljenom JavaScript skriptom, moze se izvrsiti pri pretrazi korisnika. Jedan od primera je img tag koji koristi event handler u slucaju da nema dostupne slike:

<img src = ‘x’ onerror=”console.log(‘We are inside, cookies: ‘ + document.cookies)”></img>\
\
` `S obzirom da je aplikacija lose osigurana, ovaj kod ce se izvrsiti i u konzoli dev tools ce se ispisati data poruka. Cookies u konkretnoj aplikaciji su HttpOnly pa se ne mogu dohvatiti na ovaj nacin, ali je aplikacija i dalje ranjiva na XSS napade. \
(videti slike report/XSS\_exploit, report/XSS\_before\_search, report/XSS\_after\_search, report/after\_XSS\_defence i report/cookies\_are\_http\_only)

**Odbrana od XSS napada:**

Jedan od nacina odbrane od XSS napada jeste nekoriscenje svojstva innerHTML u kodu. Preporucljivo je koristiti svojstvo textContent (dostupno kod vecine DOM elemenata), jer ono tretira tekst kao takav bez interpretacije HTML-a ili JavaScript-a.


**CSRF:**

**Primer napada:**

Testirala sam CSRF pomocu jednostave Node.js aplikacije (localhost:3000), koja ima funkciju za izmenu reda u tabeli klikom na clickbait-pehar. Zato sto u osnovnom kodu nije bilo CSRF zaštite, aplikacija je prihvatala POST zahtev iz funkcije exploit bez validacije izvora. Ovaj zahtev se izvrsio sa mojim session cookijem poslednjeg logina i uspesno izmenio red u tabeli. \
(videti sliku report/after\_csrf\_exploit)



**Odbrana od CSRF napada:**

CSRF zaštita se sprovodi korišćenjem skrivenog input polja u formi koje sadrži nasumični token generisan u klasi CsrfHttpSessionListener. Token se čuva u korisnikovoj sesiji, dohvata i proverava pri svakom POST zahtevu (npr. u metodima person i updatePerson iz person controllera). Ako token nedostaje ili se ne poklapa zahtev se odbacuje, čime se efikasno sprečavaju CSRF napadi.

**Autorizacija:**

**Primer problema:**\
Problem sa aplikacijama koje ne implementiraju autorizaciju jeste da bilo koji korisnik ulogovan na aplikaciju moze koristiti sve funkcionalnosti aplikacije. U slozenijim aplikacijama ovo vec stvara veliki problem. Cesto zelimo da imamo vodece ljude u firmama/organizacijma ili urednike odredjenih stranica, sajtova, itd. Zelimo da prosecan korisnik ovih aplikacija ima vidljiv ili dostupan samo deo sadrzaja dok visi nivoi organizacije imaju sve vise i vise ovlascenja.\
\
**Resenja problema:**\
Ovo se sprovodi matricom permisija. U njoj svaka kolona je jedna rola a svaki red jedno ovlascenje.\
Implementacija u kodu podrazumeva da baza podataka ima tabele usera, tabele rola, tabele permisija, kao i tabele koje vezuju primarne kljuceve ovih tabela. Jedan red tj. instanca nakon join-ovanja svih ovih tabela treba da za usera cuva rolu, a za rolu permisiju.\
Metodi kontrolera mogu se ogranicavati pomocu anotacije @PreAuthorize, ogranicavanje vidljivosti elemenata stranice pomocu th:disabled, a interakcija sa elementima pomocu sec:authorize. U sva tri slucaja prima se kao argument odredjena permisija koja se zahteva pre izvrsavanja/prikazivanja/pristupa.\
(videti sliku report/add\_book\_disabled, report/users\_disabled)





