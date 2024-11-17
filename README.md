### [<- Viikko 46](./viikko-46.md)

# Viikko 46
Viime viikkojen tehtävät on olleet aika tiivis paketti. Kokeillaan tällä viikolla soveltaa opittua kokonaan uuteen projektiin. Tehdään tämä 2-4 hengen ryhmissä. Jakakaa työt sellaisella tavalla, joka teistä tuntuu mielekkäältä. Tehkää kullekin tiimille oma repo GitHubiin.

## Projekti: Tuiteri

Suunnitelkaa backend, frontend sekä tietokanta alla kuvatulle sovellukselle.

### Perusominaisuudet
- Uudet käyttäjät voivat kirjautua käyttämään sovellusta
    - Käyttäjillä on **sähköpostiosoite**, uniikki **käyttäjänimi** ja **kuvaus**
- Käyttäjät voivat postata max. 144 merkin mittaisia **postauksia**
    - Postauksissa voi olla **#aihetunnisteita** ja **@viittauksia** toisiin käyttäjiin
- Käyttäjillä on oletusnäkymä, jossa näytetään sovellukseen tehtyjä postauksia. Näkymässä näytetään postauksia seuraavasti
    - Postaukset, joissa on jokin käyttäjän seuraama #aihetunniste
    - Postaukset, jotka on lähettänyt käyttäjän tykkäämä @käyttäjä
    - Postaukset, joissa on @viittaus käyttäjään
    - Postaukset näytetään lähetysjärjestyksessä, uusimmat viestit ensin
    - Käyttäjä voi merkata #aihetunnisteita seurattavaksi klikkaamalla niitä postauksista
    - Käyttäjä voi merkata @käyttäjän tykätyksi klikkaamalla tämän käyttäjänimeä postauksessa
- Käyttäjällä on näkymä, jossa hän voi tarkastella, lisätä ja poistaa seurattuja #aihetunnisteita ja tykättyjä @käyttäjiä
- Käyttäjä voi hakea postauksia #aihetunnisteen tai @käyttäjänimen perusteella
    - Käyttäjänäkymässä näytetään haetun käyttäjän profiili (käyttäjänimi, kuvaus, seurattujen käyttäjien määrä, seuraajien määrä ja kirjautumispäivämäärä) sekä käyttäjän tekemät postaukset, uusin ensin
    - Aihetunnistenäkymässä näytetään postaukset, jotka sisältävät kyseisen aihetunnisteen, uusin ensin

### Lisäominaisuudet
- Postauksiin on mahdollista lisätä kommentteja
    - Postauksen alla näytetään kuinka monta kommenttia postaus on saanut
    - Kommenttien määrää klikatessa avautuu näkymä, jossa on postaus ja sen saamat kommentit, uusin ensin
    - Kukin kommentti on itsessään kokonainen postaus kaikkine postauksen ominaisuuksineen
- Käyttäjillä on profiilikuva ja banner
    - Joka postauksessa näkyy pienenä postaajan profiilikuva
    - Käyttäjän profiilinäkymässä näytetään käyttäjän profiilikuva ja banner osana käyttäjän tietoja
- Postauksiin on mahdollista lisätä kuvia
    - Postauksen kuvat näkyvät sen tekstisisällön alla.
    - Ensimmäinen kuva näkyy isona, muut kuvat näkyvät thumbnaileina