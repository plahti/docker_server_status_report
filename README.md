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