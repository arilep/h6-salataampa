### H6 Salataampa: [Tehtävänanto](https://terokarvinen.com/linux-palvelimet/#h6-salataampa)

## Tehtävissä käytetty ympäristö

- Käyttöjärjestelmä: Windows 11 Home 64-bit
- Suoritin: AMD Ryzen 7 5800X 8-Core Processor
- Muisti: 32GB RAM
- Näytönohjain: NVIDIA GeForce RTX 3080
- Virtualisointiohjelmisto: Oracle VM VirtualBox 7.0

- Internet yhteys: DNA Netti 400M (Download 400Mbps / Upload 200Mbps)
  




## Tehtävä-X

#### _Miten Let's Encrypt -palvelu toimii_
1. Tavoitteena HTTPS-palvelimen käyttöönotto ja selainluotettavan sertifikaatin automaattinen hankinta ilman ihmisen väliintuloa.
2. Saavutetaan ajamalla sertifikaattien hallintaohjelmistoa verkkopalvelimella.
3. Hallintaohjelmisto todistaa varmentajalle (CA), että verkkopalvelin hallitsee tiettyä verkkotunnusta.
4. Agentti saa valtuutetun avainparin, jolla se voi pyytää, uusia ja peruuttaa kyseisen verkkotunnuksen sertifikaatteja.
5. CA tarkistaa, että haasteet on suoritettu oikein useista eri verkkonäkökulmista joko lataamalla tiedoston verkkopalvelimelta taikka tarkistamalla DNS-tietueen.
6. Palvelu tunnistaa palvelimen ylläpitäjän julkisen avaimen perusteella.
7. Ensimmäisellä kerralla agenttiohjelmisto luo uuden avainparin ja todistaa Let's Encryptin varmentajalle, että se hallitsee yhtä tai useampaa verkkotunnusta.
8. Tapoja, joilla agenttiohjelmisto voi todistaa verkkotunnuksen hallinnan:
    - DNS-tietueen luominen verkkotunnuksen alle
    - HTTP-resurssin lisäys tunnettuun URL-osoitteeseen
9. Näiden lisäksi Let's Encrypt antaa agentille nonce-arvon, joka agenttiohjelmiston täytyy allekirjoittaa yksityisellä avainparillaan.
10. Sertifikaatin hankkimiseksi agentti luo PKCS#10-sertifikaatin allekirjoituspyynnön (CSR) ja allekirjoittaa sen omalla yksityisellä avaimella ja valtuutetulla avainparilla.
11. CA:n myöntämä sertifikaatti tallennetaan julkisiin CT (Certificate Transparency) -lokitietoihin.
12. Sertifikaatin uusiminen tapahtuu automaattisesti valtuutetulla avainparilla.
13. Sertifikaattia peruttaessa agentti allekirjoittaa peruutuspyynnön valtuutetulla avainparilla. Let's Encrypt tarkistaa pyynnön valtuutuksen ja julkaisee peruutustiedot tavanomaisiin peruutuskanaviin (kuten OCSP-palveluun).

#### _Lego Sertifikaatin hankinta käyttäen olemassa olevaa, käynnissä olevaa verkkopalvelinta._

Voit käyttää olemassa olevaa verkkopalvelinta portissa 80 ja antaa Legon kirjoittaa tunnustiedosto HTTP-01-haasteen avaintodennuksella hakemistoon `<webroot dir>/.well-known/acme-challenge/` ja ajamalla komennon:

```
lego --accept-tos --email you@example.com --http --http.webroot /path/to/webroot --domains example.com run

```

#### _SSL konfiguraation täytyy sisältää vähintään seuraavat ohjeistukset_

```
LoadModule ssl_module modules/mod_ssl.so

Listen 443
<VirtualHost *:443>
    ServerName www.example.com
    SSLEngine on
    SSLCertificateFile "/path/to/www.example.com.cert"
    SSLCertificateKeyFile "/path/to/www.example.com.key"
</VirtualHost>
```
  
## Tehtävä-A

_Aloitin osion tekemisen 1.3.2025 klo 21:00_

Alkuun demonin potkaisu ja tarkistin että weppisivu toimii.

`sudo systemctl restart apache2`

Sivusto näytti toimivan.

Päivitykset

`sudo apt-get update`

Jonka jälkeen Legon asennus

`sudo apt-get install lego`

Tämän jälkeen hain sertifikaatin legolla testiympäristössä, komennolla:

`lego
--server=https://acme-staging-v02.api.letsencrypt.org/directory
--accept-tos
--email="ishouldchangesamplevalues@example.com"
--domains="tero.example.com" --domains="www.tero.example.com"
--http --http.webroot="/home/tero/public_sites/tero.example.com/"
--path="/home/tero/lego/certificates/"
--pem
run`

![image](https://github.com/user-attachments/assets/60a628be-8cce-4841-b1e3-167c2e7af283)

Komennon ajamisen jälkeen sertifikaatit löytyivät polusta /home/ap/lego

![image](https://github.com/user-attachments/assets/7547a146-b807-40e8-82fd-0ef5bdb1f902)

Tämän jälkeen testiserti kansio toimimattomaksi. Muutin nimeksi 'dislego'.

![image](https://github.com/user-attachments/assets/6b2475b7-01e6-41bd-8ced-36fb853aa8f8)

Ja sitten sertifikaatin hankinta komennolla:

`lego
--accept-tos
--email="ishouldchangesamplevalues@example.com"
--domains="tero.example.com" --domains="www.tero.example.com"
--http --http.webroot="/home/tero/public_sites/tero.example.com/"
--path="/home/tero/lego/certificates/"
--pem
run`

Sujui mutkitta. Polusta löytyi nyt uusi lego -hakemisto.

![image](https://github.com/user-attachments/assets/66f338d8-a7ba-4359-9d5f-416dd4bf0b48)

Tämän jälkeen otin sertifikaatin käyttöön name based virtual host -asetuksissa.

![image](https://github.com/user-attachments/assets/377eb929-6b88-4cd9-9f0c-32e4ee7e8faa)

![image](https://github.com/user-attachments/assets/5fe272a8-3743-4630-b8ed-2ec61c55fd1e)

Ja ajoin seuraavat komennot:

```
sudo systemctl restart apache2
sudo a2enmod ssl
sudo apache2ctl configtest
sudo ufw allow 443/tcp
sudo ufw enable
```

Lähti toimimaan 👍

![image](https://github.com/user-attachments/assets/63990d28-839d-4b37-913d-944f038aee5d)

Osio valmis ~21:45.

## Tehtävä-B

Testasin SSLLabs:in laadunvarmistustyökalulla oman sivustoni:

![image](https://github.com/user-attachments/assets/6dc07c23-7b2d-4d53-a8fd-0b05c95f6a9b)


## Tehtävä-C


### Lähteet

Let's Encrypt 2024: How It Works. Luettavissa: https://letsencrypt.org/how-it-works/. Luettu 1.3.20025.

Lange 2024: Lego: Obtain a Certificate. Luettavissa: https://go-acme.github.io/lego/usage/cli/obtain-a-certificate/index.html#using-an-existing-running-web-server. Luettu 1.3.20025.

The Apache Software Foundation 2025: Apache HTTP Server Version 2.4 [Official] Documentation: SSL/TLS Strong Encryption: How-To: Basic Configuration Example. Luettavissa: https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html#configexample. Luettu 1.3.2025.

Qualys SSL Labs: SSL Server Test. Luettavissa: https://www.ssllabs.com/ssltest/. Luettu 1.3.2025.

Karvinen, T. 3.12.2024. Linux Palvelimet 2025 alkukevät. Luettavissa: https://terokarvinen.com/linux-palvelimet/. Luettu 1.3.2025.
