# Server Status Report Tool

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Esittely

Server Status Report Tool on kevyt ja tehokas Python 3 -sovellus, joka on suunniteltu keräämään tietoja Linux-palvelimien tilasta ja luomaan niistä automaattisesti HTML- ja PDF-muotoisia raportteja. Raportit lähetetään kätevästi sähköpostilla, mikä mahdollistaa palvelimen tilan helpon seurannan. Työkalu on erityisen hyödyllinen Docker-kontteja käyttävissä ympäristöissä, sillä se tunnistaa automaattisesti julkaistujen porttien protokollat.

### Kerätyt tiedot:

* **Isäntänimi ja IP-osoite:** Palvelimen perusidentiteetti.
* **Muistitiedot:** Kokonaismuisti, vapaa muisti ja käytetty prosentti.
* **CPU-käyttö:** Nykyinen CPU:n käyttöaste.
* **Levytiedot:** Kytketyt levyosiot (tyyppi, koko, käytetty tila, tiedostojärjestelmä).
* **Docker-kontit:** Konttien nimet, tilat, käytetyt kuvat, julkaistut portit ja niissä tunnistetut protokollat (esim. HTTP, HTTPS, SSH, FTP, SMTP, MySQL, PostgreSQL, Redis, MongoDB, jne.).
* **Laitteistotiedot:** Sarjanumero ja malli.
* **Viimeisin päivitys:** Viimeisimmän järjestelmäpäivityksen ajankohta (`dpkg.log`-tiedostosta).
* **Järjestelmän käynnissäoloaika (Uptime):** Kuinka kauan järjestelmä on ollut käynnissä.
* **Raportin aikaleima:** Raportin luontiaika.

---

## Asennus

### Riippuvuudet

Varmista, että järjestelmääsi on asennettu seuraavat Python 3 -paketit ja niiden järjestelmätason riippuvuudet:

* `python3-jinja2`
* `python3-psutil`
* `python3-docker`
* `python3-weasyprint`

Voit asentaa nämä Debian/Ubuntu-järjestelmissä komennolla:

```bash
sudo apt update
sudo apt install python3-jinja2 python3-psutil python3-docker python3-weasyprint

Jos python3-weasyprint -pakettia ei löydy suoraan tai sen asennuksessa on ongelmia riippuvuuksien kanssa, voit kokeilla sen järjestelmätason riippuvuuksien asennusta ja sen jälkeen pip-asennusta:

sudo apt install python3-lxml python3-cffi libcairo2 libpango-1.0-0 libpangocairo-1.0-0 libgdk-pixbuf2.0-0 libffi-dev shared-mime-info
pip3 install weasyprint

Lisäksi, varmista että Docker-palvelu on käynnissä ja käyttäjällä, jolla skriptiä ajetaan, on oikeudet Docker-demoniin (yleensä lisäämällä käyttäjä docker-ryhmään ja käynnistämällä sessio uudelleen):

sudo usermod -aG docker $USER
# Kirjaudu ulos ja takaisin sisään tai käynnistä järjestelmä uudelleen, jotta ryhmämuutos tulee voimaan.

Käyttöönotto ja konfiguraatio
Luo tarvittavat hakemistorakenteet:

sudo mkdir -p /etc/serverstatusreport
sudo mkdir -p /usr/share/serverstatusreport

Sijoita Python-skripti: Kopioi server_status_report.py -skripti ja aseta sille suoritusoikeudet.

sudo cp server_status_report.py /usr/local/bin/server_status_report
sudo chmod +x /usr/local/bin/server_status_report

Luo HTML-raporttimalli: Sijoita html_template.html -tiedosto (TEMPLATE_PATH) seuraavaan sijaintiin.

sudo cp html_template.html /usr/share/serverstatusreport/

Huom: Sinun tulee luoda tai päivittää tämä HTML-malli vastaamaan kerättyjä tietoja, erityisesti laajennettuja Docker-konttien protokollatietoja (tunnistettu protokolla, banneri jne.).

Konfiguroi sähköpostiasetukset: Luo tiedosto /etc/serverstatusreport/email_config.json seuraavalla sisällöllä. Vaihda tiedot omiin asetuksiisi.

{
    "from_email": "raportti@esimerkki.com",
    "to_email": "vastaanottaja@esimerkki.com",
    "smtp_server": "smtp.esimerkki.com",
    "port": 587,
    "username": "raportti@esimerkki.com",
    "password": "salasana_tähän"
}

Turvallisuusvinkki: Salasanojen säilyttäminen selväkielisenä tiedostossa ei ole ihanteellista turvallisuuden kannalta. Harkitse ympäristömuuttujien tai salatun keyring-järjestelmän käyttöä tuotantoympäristössä. Muista rajoittaa tiedoston oikeudet:

sudo chmod 600 /etc/serverstatusreport/email_config.json

Määritä porttien polut (valinnainen): Jos Docker-konttien julkaistut HTTP/HTTPS-palvelut vaativat tietyn URL-polun (esim. http://ip:port/polku), voit määrittää nämä tiedostossa /etc/serverstatusreport/port_paths.txt. Muoto on portti;polku.

8080;/app
9000;/admin

Ajaminen
Voit ajaa skriptin manuaalisesti terminaalissa:

sudo /usr/local/bin/server_status_report

Skripti luo HTML- ja PDF-raportit /tmp/server_status_report.html ja /tmp/server_status_report.pdf -tiedostoihin ja lähettää ne sähköpostitse, jos sähköpostikonfiguraatio on kunnossa.

Cron-ajastus
Suositeltavaa on ajaa raportti säännöllisesti cron-tehtävänä. Voit avata cron-editorin komennolla:

sudo crontab -e

Lisää sitten rivi, joka ajaa skriptin esimerkiksi kerran päivässä klo 03:00. Tämä komento ohjaa kaiken tulosteen ja virheet /dev/null-tiedostoon, jotta cron ei lähetä sinulle sähköpostia jokaisesta suorituksesta. Lokiviestit menevät edelleen järjestelmän lokiin (esim. journalctl -f tai /var/log/syslog).

0 3 * * * /usr/local/bin/server_status_report >/dev/null 2>&1

.deb-paketin rakentaminen
Tämä osio opastaa sinua luomaan Debian-paketin ( .deb ) sovelluksesta. Oletamme, että sinulla on kaikki tarvittavat lähdetiedostot (Python-skripti, HTML-malli) valmiina.

Esivaatimukset
Tarvitset dpkg-dev-paketin, joka sisältää dpkg-deb-työkalun:

sudo apt install dpkg-dev

Paketin tiedostorakenne
Luo seuraavanlainen hakemistorakenne paketin rakentamista varten. Korvaa serverstatusreport_1.0.0 omalla paketin nimellä ja versiolla (tämän kansion nimi on tärkeä dpkg-deb-komennolle):

serverstatusreport_1.0.0/
├── DEBIAN/
│   └── control
├── etc/
│   └── serverstatusreport/
│       ├── email_config.json  # Konfiguraatiotiedosto, joka EI saa ylikirjoittua päivityksessä
│       └── port_paths.txt     # Konfiguraatiotiedosto, joka EI saa ylikirjoittua päivityksessä
├── usr/
│   ├── local/
│   │   └── bin/
│   │       └── server_status_report  # Python-skripti
│   └── share/
│       └── serverstatusreport/
│           └── html_template.html    # HTML-malli

Kopioi server_status_report.py tiedosto polkuun serverstatusreport_1.0.0/usr/local/bin/server_status_report ja html_template.html tiedosto polkuun serverstatusreport_1.0.0/usr/share/serverstatusreport/html_template.html.

DEBIAN/control -tiedosto
Tämä on kriittinen tiedosto, joka sisältää paketin metatiedot ja riippuvuudet. Muista, että tässä tiedostossa ei saa olla kommentteja (rivit, jotka alkavat #) ja muotoilun on oltava erittäin tarkka (tarkalleen yksi välilyönti Conffiles:-kentän alla olevien polkujen edessä ja yksi tyhjä rivi tiedoston lopussa).

Kopioi tämä sisältö tarkasti serverstatusreport_1.0.0/DEBIAN/control -tiedostoon:

Package: serverstatusreport
Version: 1.0.0
Section: utils
Priority: optional
Architecture: all
Depends: python3, python3-jinja2, python3-psutil, python3-docker, python3-weasyprint
Maintainer: petri.lahti@iki.fi
Description: Palvelimen tilaraporttityökalu, joka tuottaa HTML+PDF-raportin ja lähettää sen sähköpostilla.
Conffiles:
 /etc/serverstatusreport/port_paths.txt
 /etc/serverstatusreport/email_config.json


HUOM: Varmista, että /etc/serverstatusreport/email_config.json -rivin jälkeen on tarkalleen yksi tyhjä rivi.

Paketin rakentaminen
Kun tiedostorakenne ja control-tiedosto ovat kunnossa, siirry hakemistoon, joka sisältää serverstatusreport_1.0.0/ -kansion (ei sen sisälle!). Suorita sitten dpkg-deb -komento:

dpkg-deb --build serverstatusreport_1.0.0/

Jos kaikki meni oikein, komento luo .deb-paketin samaan hakemistoon (esim. serverstatusreport_1.0.0.deb).

Paketin asentaminen
Voit asentaa luodun .deb-paketin komennolla:

sudo dpkg --install serverstatusreport_1.0.0.deb

Tärkeää: Kun asennat paketin tällä tavalla, dpkg havaitsee Conffiles-kentän ansiosta, että /etc/serverstatusreport/email_config.json ja port_paths.txt ovat konfiguraatiotiedostoja. Jos olet muokannut niitä asennuksen jälkeen, dpkg kysyy, haluatko säilyttää omat muutoksesi vai korvata ne paketin mukana tulevalla versiolla.

Jos saat asennuksessa virheitä puuttuvista riippuvuuksista (esim. python3-weasyprint), voit korjata ne ajamalla:

sudo apt --fix-broken install

Kehitys ja parannusehdotukset
Tässä on joitakin ideoita tuleviin parannuksiin, joita voidaan harkita:

Joustavampi konfiguraatio: Kaikkien asetusten (polut, sähköposti, protokollatunnistus) siirtäminen yhteen YAML/TOML-tiedostoon, jota ohjelma lukee.

Kehittyneempi lokitus: Mahdollisuus asettaa lokituksen tasoa komentoriviargumenteilla (esim. -v tai --verbose).

Testaus: Yksikkötestien ja integraatiotestien lisääminen koodin luotettavuuden varmistamiseksi.

Responsiivinen HTML-raportti: Paranna raportin ulkoasua CSS:n ja JavaScriptin avulla, jotta se näyttää hyvältä eri laitteilla ja sisältää ehkä jopa yksinkertaisia kaavioita.

Laajemmat järjestelmätiedot: Lisää verkkoliikenteen tilastoja tai top-listoja resursseja kuluttavista prosesseista.

Turvallisempi sähköpostin tunnusten käsittely: Vältä salasanojen säilyttämistä selväkielisenä tiedostossa.

Lisenssi
Tämä projekti on lisensoitu MIT-lisenssillä. Katso lisätietoja LICENSE-tiedostosta (jos sellainen on).