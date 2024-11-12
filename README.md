### [<- Viikko 45](./viikko-45.md)
# Viikko 46
## Maanantai
### Frontend

Tiedon välittämiseen komponentilta toiselle käytetään Reactissa **propseja** ja **eventtejä**, mutta joskus jotain tietoa tarvitaan joko niin kaukana sitä säilyttävästä komponentista, tai niin monessa paikkaa sovelluksessa, että tämä voi johtaa ns. *prop drilling*-ilmiöön, jossa tietoa joudutaan passaamaan pitkien komponenttihierarkioiden läpi. Tällaisia tilanteita varten Reactissa on [Context API](https://react.dev/learn/passing-data-deeply-with-context). Konteksti on kätevä tapa hallita esim. väriteemaa, kirjautumista ja vaikkapa ostoskoria.

### FE-Tehtävä

Lisäsimme viime viikolla tilavarauskantaan **kayttajat**-taulun, jossa on varausjärjestelmän käyttäjien tietoja. Lisätään tauluun kenttä **rooli**, jonka arvo on joko `"user"` tai `"admin"`. Meillä pitäisi myös olla viime viikolla toteutettu http-backend, jossa on endpoint kirjautumista varten.

Toteutetaan varausjärjestelmän frontendiin lomake, joka kysyy käyttäjänimeä ja salasanaa ja lähettää ne kirjautumis-endpointtiin. Backend tarkastaa ovatko kirjautumistiedot valideja, ja jos ovat, palauttaa frontille käyttäjän tiedot sekä istunnon tunnuksen tai tokenin. Frontend tallettaa tiedon kirjautumisesta kontekstiin, ja antaa sen perusteella käyttäjälle mahdollisuuden päästä käsiksi eri toimintoihin:

- ilman kirjautumista käyttäjä ei pääse järjestelmään ollenkaan
- user-roolin käyttäjä voi tarkastella kaikkia tietoja ja luoda uusia varauksia
- admin-roolin käyttäjä voi muokata myös tiloja ja käyttäjiä.

Frontend myös huolehtii siitä, että tunnistautuminen toteutetaan niin kuin backend sitä tarvitsee. Jos backend asettaa tokenin tai istuntotunnuksen evästeeseen, selain palauttaa sen automaattisesti, eikä frontin tarvitse käsitellä niitä erikseen. API-kutsuun käytettävä kirjasto tosin vaatii yleensä **credentials** -asetuksen, esim:

```JavaScript
fetch('/api/endpoint', {
  method: 'GET',
  credentials: 'include'
});
```

tai:

```JavaScript
axios.get('/api/endpoint', { withCredentials: true });
```

Jos backend haluaa tokenin **Authorization** -headerissa, se täytyy asettaa frontissa käsin, esim:

```JS
fetch('/api/endpoint', {
  method: 'GET',
  headers: {
    'Content-type': 'application/json',
    'Authorization': `Bearer ${token}`
  }
});
```

tai:

```js
axios.get('/api/endpoint', {
    headers: {
        'Content-type': 'application/json',
        'Authorization': `Bearer ${token}`
    }
})
```

Voit tehdä tämän tehtävän myös yhteistyössä jonkun BE-ryhmäläisen kanssa niin, että käytät hänen tekemäänsä backendiä.

### FE resursseja
- https://dev.to/miracool/how-to-manage-user-authentication-with-react-js-3ic5
- https://medium.com/@yogeshmulecraft/implementing-protected-routes-in-react-js-b39583be0740

### Backend
Django sisältää valmiit toiminnallisuudet autentikointiin ja auktorisointiin: `User`-mallin, `LoginView`:n ja `LogoutView`:n. Django käyttää autentikointiin oletusarvoisesti istuntoja. DRF laajentaa tätä pohjaa lisätoiminnallisuuksilla kuten `SessionAuthentication-` ja `TokenAuthentication`-luokilla. DRF:ssä ei ole valmista tukea JWT:lle, mutta siihen voidaan käyttää kolmannen osapuolen kirjastoa kuten [simplejwt](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/).

Haluttu autentikointimetodi määritellään Djangon `settings.py`:ssä, esim:
```Python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}
```

Ja tokenien luomiseen ja virkistämiseen voidaan käyttää esim. `simplejwt`:n sisäänrakennettuja endpointteja määrittelemällä niille polut projektin `urls.py`:ssä seuraavasti:

```Python
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    # other urls
]
```

Endpointin voi suojata käyttämällä DRF:n `permission_classes`-dekoraattoria esim. seuraavasti:

```Python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def protected_view(request):
    return Response({"message": "This is a protected view accessible only with a valid token."})
```

### BE-Tehtävä
Luo `simplejwt`-kirjastoa käyttävä autentikointi ja auktorisointi, joka sallii user-rooliin kuuluvien käyttäjien tarkastella käyttäjiä, tiloja ja varauksia, ja luoda uusia varauksia. Admin-rooliin kuuluvat käyttäjät saavat täydet CRUD-oikeudet kaikkiin tauluihin.

### BE-Resursseja
- https://www.django-rest-framework.org/api-guide/authentication/
- https://www.django-rest-framework.org/api-guide/permissions/
- https://django-rest-framework-simplejwt.readthedocs.io/en/latest/

## Tiistai 
### Frontend
Tiedon hakemisen lisäksi tietoa pitää usein voida myös lähettää frontista REST-rajapintaan. Reactilla voidaan toki poimia lähetettävät tiedot mistä hyvänsä kentistä ja liipaista http-kysely millä eventillä halutaan, mutta yleensä on mielekkäintä hyödyntää HTML-lomakkeita. Selaimissa on paljon valmiita toiminnallisuutta rakennettuna HTML:n `<forms>`-elementin ja kenttien kuten `<input>`, `<select>` ja `<textarea>` ympärille. Lisäksi lomakkeiden käsittelyä helpottavat kirjastot käyttävät juuri näitä elementtejä lähtökohtana.

Perusmuodossaan voimme luoda JSX:llä lomakkeen, esim:
```JSX
<form onSubmit={handleSubmit}>
    <div>
        <label htmlFor="username">Username:</label>
        <input type="text" id="username" name="username" required />
    </div>
    <div>
        <label htmlFor="password">Password:</label>
        <input type="password" id="password" name="password" required />
    </div>
    <button type="submit">Login</button>
</form>
```

Tämä lomake liipaisee `handleSubmit`-funktion, kun **login** -painiketta painetaan. Funktiossa voimme poimia kenttiin syötetyt arvot lomakkeesta ja lähettää ne endpointille:

```JS
async function handleSubmit(e) {
    // Estetään sivun virkistäminen, joka on oletustoiminnallisuus, kun lomakkeen lähetyspainiketta painetaan
    e.preventDefault();

    // Poimitaan kenttien tiedot lomakkeesta (joka on event-objektin target)
    const formData = new FormData(e.target);
    const username = formData.get('username');
    const password = formData.get('password');

    // Kutsutaan endpointtia kenttien arvoilla
    fetch('/api/login', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({ username, password }),
    })
        .then(res => res.json())
        .then(data => console.log('data', data))
        .catch(error => console.error('virhe:', error));
}
```

Usein haluamme kuitenkin esim. käsitellä lomakkeen ja kenttien tiloja (muokattu, lähetetty, validoitu, lähetty) tarkemmin frontilla, tai tarvitsemme tavallista monimutkaisempia validointisääntöjä tai dynaamisia kenttiä. Lomakkeiden käsittelyyn on olemassa omia kirjastoja, joilla nämä ja monet muut erikoistilanteet on paljon helmpompi käsittellä. Käytetään näissä harjoituksissa [React Hook Form](https://react-hook-form.com/get-started):ia.

Lyhyesti kuvaten RFH:ta käytetään niin, että lomakekomponentissa destrukturoimme `useForm`-hookista halutut metodit ja propertyt:

```JS
  const { register, handleSubmit, watch, formState: { errors } } = useForm<Inputs>()
```

JSX:ssä lomakkeen `onSubmit`-käsittelijälle annetaan useFormista purettu `handleSubmit`, ja sille annetaan parametriksi meidän oma funktio, jolla lähetys käsitellään. Kentät rekisteröidään `register`-metodilla, ja niille voidaan konfiguroida erilaisia validointisääntöjä, joiden virheet React Hook Form päivittää dynaamisesti `formStaten` `errors` -propertyyn:

```JSX
<form onSubmit={handleSubmit(onSubmit)}>
    <label htmlFor="username">Username</label>
    <input id="username" {...register('username', { required: 'Username is required' })} /><br>
    {errors.username && <p>{errors.username.message}</p>}

    <label htmlFor="password">Password</label>
    <input type="password" id="password" {...register('password', { required: 'Password is required' })} /><br>
    {errors.password && <p>{errors.password.message}</p>}
    
    <button type="submit">Login</button>
</form>
```

RHF:n handleSubmit purkaa kenttien arvot meille valmiiksi paketiksi, mikä vähentää boilerplatea lähetyksen käsittelyfunktiosta:
```JS
    function onSubmit(data) {
        fetch('/api/login', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(data),
        })
        .then(res => res.json())
        .then(data => console.log('data', data))
        .catch(error => console.error('virhe:', error));
    }
```

### FE-tehtävä

Asenna FE-ympäristöön React Hook Form komennolla `npm install react-hook-form`. Päivitä tilavaraus-frontin lomakkeet käyttämään React Hook Formia. Lisää lomakkeisiin mielekkäitä validointisääntöjä (pakollisuuksia, pituusrajoituksia, minimi- ja maksimiarvoja ja patterneja) kokeilemalla eri vaihtoehtoja, ks. https://react-hook-form.com/docs/useform/register 

### FE-resursseja
- https://react-hook-form.com/
- https://medium.com/@wteja/mastering-form-validation-with-react-hook-form-a-comprehensive-guide-5c63a5efaeab

### Backend
Django REST Frameworkin tarjoilemat tiedot määritellään [tietomallien](https://www.django-rest-framework.org/tutorial/1-serialization/#creating-a-model-to-work-with) avulla, ei niinkään suoraan muokkaamalla tietokantaa. Tietomalleilla määritellään taulut, niiden kentät, kenttien ominaisuudet ja taulujen relaatiot objektimuodossa instansoimalla `models.Model`-luokkaa esim. seuraavasti:

```Python
from django.db import models

class Varaus(models.Model):
    varaaja = models.ForeignKey(Varaajat, on_delete=models.CASCADE)
    tila = models.ForeignKey(Tilat, on_delete=models.CASCADE)
    varauspaiva = models.DateField()
```

Huomaa, että mallissa ei tarvitse erikseen määrittää juoksevaa id-kenttää. DRF:n tietomallit saavat sen automaattisesti.

Tietomallien pohjalta tehdään [serialisoijat](https://www.django-rest-framework.org/tutorial/1-serialization/#creating-a-serializer-class), joiden avulla kannasta haetut tiedot voidaan muotoilla halutun muotoiseksi JSON-objektiksi ja päinvastoin. Mallin pohjalta serialisoija määritellään käyttämällä `ModelSerializer`-luokkaa:

```Python
from rest_framework import serializers

class VarausSerializer(serializers.ModelSerializer):
    class Meta:
        model = Varaus
        fields = ['varaaja', 'tila', 'varauspaiva']
```

Malleja ja serialisoijia käyttäen voidaan luoda [näkymiä](https://www.django-rest-framework.org/tutorial/2-requests-and-responses/#wrapping-api-views) (views), jotka ovat funktioita tai luokkia, jotka vastaanottavat HTTP-pyynnön ja tuottavat siihen vastauksen. Näkymät voi luoda käsin, mutta DRF:ssa on myös kätevä abstraktio [ViewSet](https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/#tutorial-6-viewsets-routers), joka tuottaa mallin ja serialisoijan pohjalta tyypilliset CRUD-toiminnot:

```Python
from rest_framework import viewsets
from .models import Varaus
from .serializers import VarausSerializer

class VarausViewSet(viewsets.ModelViewSet):
    queryset = Varaus.objects.all()
    serializer_class = VarausSerializer
```

ViewSetin näkymät kartoitetaan polkuihin tähän tyyliin:

```Python
path('varaukset/', VarauksetViewSet.as_view({'get': 'list', 'post': 'create'}), name='varaukset'),
```

**HUOM:** Aina kun tietomalleihin tulee muutoksia, täytyy ajaa [migraatiot](https://docs.djangoproject.com/en/5.1/topics/migrations/), jotka päivittävät muutokset kantaan. Tämä tehdään seuraavilla komennoilla:

```
python manage.py makemigrations
python manage.py migrate
```

### BE-tehtävä
Aloita tarvittaessa uusi DRM-projekti, ja luo siihen tilavarausjärjestelmän tarvitsemat tietomallit ja niille API-näkymät. Kuvaukset tauluista ja kentistä löytyvät [viikon 44 tehtävistä](viikko-44.md#keskiviikko). Ota myös autentikointi ja auktorisointi käyttöön eilisen tehtävän mukaisesti.

### BE-resursseja
- https://www.django-rest-framework.org/api-guide/serializers/
- https://www.django-rest-framework.org/tutorial/1-serialization/#creating-a-model-to-work-with
- https://www.django-rest-framework.org/tutorial/1-serialization/#creating-a-serializer-class


## Keskiviikko 
### Frontend

Olet jo törmännytkin Reactin [koukkuihin](https://react.dev/reference/react/hooks) (hooks). Koukut toivat funktionaalisiin komponentteihin mahdollisuuden käsitellä komponentin tilaa ja sivuvaikutuksia, sekä kytkeä toiminnallisuuksia komponentin elämänkaaritapahtumiin. Olennaisimmat Reactin sisäänrakennetut koukut ovat:
- [useState](https://react.dev/reference/react/useState), jolla voidaan luoda ja käsitellä komponentin tilaa
- [useEffect](https://react.dev/reference/react/useEffect), jolla voidaan liipaista toimintoja, kun komponentti piirretään tai joku tieto sen tilassa muuttuu
- [useContext](https://react.dev/reference/react/useContext), joka antaa mahdollisuuden luoda koko sovelluksen läpäisevän kontekstin, jossa olevaan tietoon komponentit pääsevät helposti käsiksi

Koukkuja on myös mahdollista [luoda itse](https://react.dev/learn/reusing-logic-with-custom-hooks). Tämä voi olla hyvä keino luoda modulaarista, uudelleenkäytettävää ja ylläpidettävää koodia.

Koukku on lyhyesti sanoen funktio, joka alkaa etuliitteellä **use**, ja sisältää jotain logiikkaa, jota halutaan käyttää useammassa kohtaa sovellusta. Esimerkiksi voimme tehdä koukun, joka tarkkailee selainikkunan koon muutoksia, nimeltä `useWindowSize`:

```JS
import { useState, useEffect } from 'react';

function useWindowSize() {
    const [windowSize, setWindowSize] = useState({
        width: window.innerWidth,
        height: window.innerHeight,
    });

    useEffect(() => {
        function handleResize() {
            setWindowSize({
                width: window.innerWidth,
                height: window.innerHeight,
            });
        }

        window.addEventListener('resize', handleResize);
        handleResize();
        return () => window.removeEventListener('resize', handleResize);
    }, []);

    return windowSize;
}

export default useWindowSize;
```

Koukku palauttaa tilamuuttujan `windowSize`, jonka arvona on objekti, joka sisältää ikkunan leveyden ja korkeuden. Koukkua voidaan käyttää toisessa komponentissa esimerkiksi näin:

```JS
import React from 'react';
import useWindowSize from './useWindowSize';

function MyComponent() {
    const { width, height } = useWindowSize();

    return (
        <div>
            <h2>Ikkunan koko:</h2>
            <p>Leveys: {width}px</p>
            <p>Korkeus: {height}px</p>
        </div>
    );
}

export default MyComponent;
```

### FE-tehtävä

Perehdy Reactin sisäänrakennettuihin koukkuihin. Olemme jo käyttäneet niistä useita, joten voit palata jo aiemmin kijoitettuun koodiin ja katsoa uusin silmin, miten niitä on sovellettu. Perehdy custom-koukkuihin, ja toteuta koukku, jolla haetaan tietoa APIsta. Koukku ottaa parametriksi **url**:n, ja palauttaa haetun **datan**, totuusarvon **loading**, joka kertoo onko lataaminen vielä käynnissä, sekä merkkijonon **error**, joka sisältää mahdollisen virheilmoituksen.

### FE-resursseja
- https://react.dev/reference/react/hooks
- https://react.dev/reference/react/useState
- https://react.dev/reference/react/useEffect
- https://react.dev/reference/react/useContext
- https://react.dev/learn/reusing-logic-with-custom-hooks

### Backend
Django Signals on Djangon sisäänrakennettu viestitoiminnallisuus, joiden avulla tietyt toiminnot, kuten http-kutsun vastaanottaminen tai tiedon poistaminen, voivat liipaista lisätoimintoja. Niiden avulla voidaan esimerkiksi automatisoida tervetulosähköpostin lähettäminen, kun uusi käyttäjä kirjautuu järjestelmään, notifikaation esittäminen kun tiettyyn viestiin tulee kommentti, tai palvelimen toimintojen kirjaaminen lokitiedostoon.

Signaalit käyttävät julkaisija-tilaaja (publisher-subscriber) -mallia, jossa lähettäjä lähettää signaalin, ja sen tilanneet vastaanottajat suorittavat sen saadessaan jonkin toiminnon.

Signaali määritellään instansoimalla `django.dispatch.Signal` -luokka, ja se liitetään vastaanottajiin `signal.connect()` -metodilla.

### BE-tehtävä

Perehdy Djangon signaaleihin, ja toteuta sovellukseen toiminnallisuus, jossa epäonnistuneet sisäänkirjautumiset kirjataan lokitiedostoon.

### BE-resursseja
- https://www.sitepoint.com/understanding-signals-in-django/
- https://medium.com/jungletronics/how-django-signals-work-81dc30d0dad5

## Torstai 
### Frontend

Kaikkien React-komponenttien tekeminen itse HTML:stä on täysin mahdollista, mutta HTML:n oletusmuotoilut ovat aika karuja, ja yhtenäisen visuaalisen ilmeen tyylittely kokonaiselle sovellukselle on iso urakka. Järkevä vaihtoehto itse tekemiselle on käyttää jotain valmista UI-kirjastoa. Perehdymme tässä [Material UI (MUI)-kirjastoon](https://mui.com/material-ui/getting-started/), joka on avoimen lähdekoodin implementaatio Googlen samannimisestä visuaalisesta ohjeistosta. 

MUI asennetaan komennolla `npm install @mui/material @emotion/react @emotion/styled`, joka asentaa MUI:n sekä sen tyylittelyyn käytettävän Emotion-moottorin. Lisäksi on suositeltavaa asentaa kirjaston käyttämä Roboto-fontti: `npm install @fontsource/roboto`, joka pitää lisäksi ottaa käyttöön lisäämällä seuraavat rivit React-projektin juuritiedostoon (`main.jsx`):
```JS
import '@fontsource/roboto/300.css';
import '@fontsource/roboto/400.css';
import '@fontsource/roboto/500.css';
import '@fontsource/roboto/700.css';
```
Voit myös asentaa Material Icons -ikonisetin komennolla `npm install @mui/icons-material`. Voit tarkastella settiin kuuluvia ikoneita osoitteessa https://mui.com/material-ui/material-icons/. 

MUI:ta käytetään niin, että HTML-elementtien sijaan (tai niiden ohessa) käytetään MUI:n valmiiksi muotoiltuja komponentteja. Useimmille sisältötageille löytyy MUI:sta toimiva vastine, ja niiden lisäksi kirjasto sisältää monenlaisia valmiita palikoita eri tarkoituksiin, kuten navigaatiopalkkeja, sivusta aukeavia menuja, notifikaatioita ja paljon muuta.

### FE-tehtävä

Perehdy Material UI:n tarjontaan sen [sivustolla](https://mui.com/material-ui/all-components/). Perehdy erityisesti tässä vaiheessa [tekstikenttään](https://mui.com/material-ui/react-text-field/), [painikkeeseen](https://mui.com/material-ui/react-button/), [tauluun](https://mui.com/material-ui/react-table/), [containeriin](https://mui.com/material-ui/react-container/) sekä [yläpalkkiin](https://mui.com/material-ui/react-app-bar/).

Otetaan työn alle tuttu tilavaraussovellus. Voit aloittaa kokonaan uuden projektin MUI-muotoiluilla, mutta MUI:n asentaminen olemassaolevaan projektiin voi olla havainnollisempaa. Voit säilyttää sovelluksen logiikan samanlaisena kuin se nyt on, ja vain korvata JSX:ssä olevia HTML-elementtejä vastaavilla MUI-komponenteilla, ja katsoa miltä ne näyttävät ja miten ne toimivat osana sovellusta.

Tehtävänä on päivittää tilavaraussovellus käyttämään MUI:ta: sijoita navigaatio yläpalkkiin, ja korvaa eri näkymissä olevat taulut ja lomakkeet MUI:n komponenteilla.

### FE-resursseja
- https://mui.com/material-ui/getting-started/
- https://mui.com/material-ui/all-components/
- https://mui.com/material-ui/material-icons/
- https://www.youtube.com/watch?v=0KEpWHtG10M&list=PL4cUxeGkcC9gjxLvV4VEkZ6H6H4yWuS58

### Backend
Lähes aina meillä täytyy olla valmius jollain tavoin rajoittaa endpointista tarjoiltavan datan määrää. Useimmissa järjestelmissä tietokantoihin kertyy tuhansia tai jopa miljoonia rivejä, ja niiden kaikkien tarjoaminen frontille kerralla ei ole mielekästä. Tyypillisimmät tavat rajoittaa haettavaa dataa ovat [sivutus](https://www.django-rest-framework.org/api-guide/pagination/) (pagination) ja [suodatus](https://www.django-rest-framework.org/api-guide/filtering/) (filtering).

**Sivutuksessa** määritellään tietty maksimimäärä rivejä, joita haetaan kerrallaan. API:n käyttäjä asettaa url-parametrit, jotka kertovat joko haettavan [sivunumeron](https://www.django-rest-framework.org/api-guide/pagination/#pagenumberpagination), tai haettavien rivien määrän ja haun alkupisteen [(limit ja offset)](https://www.django-rest-framework.org/api-guide/pagination/#limitoffsetpagination). Kolmas DRF:n oletuksena tarjoama sivutusmetodi on [kursorisivutus](https://www.django-rest-framework.org/api-guide/pagination/#cursorpagination), jossa on mahdollista navigoida vain eteen- ja taaksepäin, mutta ei siirtyä satunnaiselle sivulle listassa. Kursorisivutuksen hyöty on, että se toimii suorituskykyisesti myös erittäin suurissa tauluissa, joissa muunlaiset sivutukset voivat tukehtua.

**Suodatuksessa** määritetään parametri tai parametreja, joiden perusteella osa riveistä jätetään näyttämättä. Esimerkiksi voimme rajoittaa tilavarauspalvelun tilanäkymää niin, että se näyttää vain tilat, joiden nimessä on merkkijono "A1", tai varausnäkymää niin, että siinä näytetään vain varaukset, jotka kohdistuvat joulukuuhun, ja jotka on tehnyt Erkki Merkkinen.

Yksinkertaisia suodatuksia voi DRF:ssä tehdä yliajamalla `GenericAPIView`:n `get_queryset()`-metodia niin, että rajoittaa hakua esimerkiksi HTTP-kutsussa annetun parametrin perusteella käyttämällä Djangon Queryset-APIn [kenttähakuja](https://docs.djangoproject.com/en/5.1/ref/models/querysets/#id4) (field lookups) esimerkiksi näin:

```Python
from myapp.models import Purchase
from myapp.serializers import PurchaseSerializer
from rest_framework import generics

class TilaList(generics.ListAPIView):
    serializer_class = TilaSerializer

    def get_queryset(self):
        queryset = Tila.objects.all()
        varaaja = self.request.query_params.get('varaaja')
        if varaaja is not None:
            queryset = queryset.filter(tila__varaaja=varaaja)
        return queryset
```

Jos tarvitaan monimutkaisempia suodatuksia, voi olla järkevää asentaa `django-filter`-kirjasto, joka yksinkertaistaa ja standardoi DRM:n suodatuksia. Kirjasto asennetaan komennolla `pip install django-filter`, ja konfiguroidaan käyttöön `settings.py`:ssä seuraavasti:

```Python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend'],
}
```

`Django-filter`-kirjaston tarjoamaa geneeristä suodatusta käytetään luomalla tietomallin pohjalta `FilterSet`:
```Python
import django_filters
from .models import Varaus

class VarausFilter(django_filters.FilterSet):
    varaaja = django_filters.CharFilter(lookup_expr='icontains')
    tila = django_filters.CharFilter(lookup_expr='icontains')
    varauspaiva = django_filters.DateFromToRangeFilter()

    class Meta:
        model = Varaus
        fields = ['varaaja', 'varaus', 'varauspaiva']
```

Tässä määritellään, että `varaaja`- ja `tila`-kenttiä voidaan suodattaa osittaisella merkkijonolla, jonka kirjainkokoa ei huomioida. `Varauspaiva`-kenttää voidaan suodattaa antamalla alku- ja/tai loppupäivämäärä.

Suodatin otetaan käyttöön näkymässä seuraavasti:
```Python
from rest_framework.generics import ListAPIView
from .models import Varaus
from .serializers import VarausSerializer
from .filters import VarausFilter

class VarausListView(ListAPIView):
    queryset = Varaus.objects.all()
    serializer_class = VarausSerializer
    filterset_class = VarausFilter
```

Tätä näkymää voidaan suodattaa url-parametreilla, esim. `?varaaja=kalle&varauspaiva_after=2024-12-10`.

### BE-tehtävä

Toteuta tilavaraussovelluksen tauluihin suodatukset kaikkien kenttien pohjalta sekä sivutus niin, että yhdelle sivulle mahtuu 10 riviä. Lisää tarvittaessa kantaan rivejä, jotta voit testata suodatusta ja sivutusta.

### BE-resursseja
- https://www.django-rest-framework.org/api-guide/pagination/
- https://www.django-rest-framework.org/api-guide/filtering/
- https://docs.djangoproject.com/en/5.1/ref/models/querysets/#id4
- https://django-filter.readthedocs.io/en/stable/guide/rest_framework.html

## Perjantai
### Frontend 

MUI:n komponenttien ulkoasua määrittää **teema**, joka sisältää mm. fonttiasetukset, tekstikoot eri tilanteissa, väripaletin, pudotusvarjojen asetukset jne. Teema syötetään sovellukseen käyttämällä `ThemeProvider`-komponenttia, joka käyttää pellin alla Reactin **kontekstia.** Teeman avulla sovelluksen komponentit noudattelevat yhtenäistä visuaalista linjaa. 

Teemaa on mahdollista muokata niin, että se vastaa esimerkiksi yrityksen graafista ohjeistoa. Yksi keino tähän on muokata [teemamuuttujia](https://mui.com/material-ui/customization/theming/#theme-configuration-variables) luomalla teemaobjekti `createTheme()`-metodilla, antamalla sille parametrina muutokset oletusteemaan verrattuna, ja tarjoilemalla teema `ThemeProviderilla` esim. seuraavasti:

```JS
import * as React from 'react';
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { lime, purple } from '@mui/material/colors';
import Button from '@mui/material/Button';

const theme = createTheme({
  palette: {
    primary: {
      main: '#FF5733',
      // light: lasketaan palette.primary.main:in perusteella,
      // dark: lasketaan palette.primary.main:in perusteella,
      // contrastText: lasketaan niin, että se erottuu hyvin palette.primary.main:iä vasten
    },
    secondary: {
      main: '#E0C2FF',
      light: '#F5EBFF',
      // dark: lasketaan palette.secondary.main:in perusteella,
      contrastText: '#47008F',
    },
  },
});

export default function UsingColorObject() {
  return (
    <ThemeProvider theme={theme}>
      <Button variant="contained">Primary</Button>
      <Button variant="contained" color="secondary" sx={{ ml: 2 }}>
        Secondary
      </Button>
    </ThemeProvider>
  );
}
```
Tässä muokataan teeman primaarivärin `main`-tokenia (kirkkaan ja pimeän teeman variantit sekä sopiva tekstiväri generoidaan automaattisesti), sekä sekundaarivärin `main`-, `light`- ja `contrastText`-tokeneita (pimeän teeman variantti generoidaan automaattisesti). `ThemeProvider`:in sisällä olevat MUI-komponentit käyttävät sitten näitä värejä.

Jos luot omia custom-komponentteja, voit tehdä ne seuraamaan voimassaolevaa MUI-teemaa käyttämällä `styled()`-wrapperia MUI-sivuston [ohjeiden](https://mui.com/material-ui/customization/creating-themed-components/) mukaan. Ratkaisu vaatii vähän boilerplate-koodia; alla esimerkki komponentista, joka koostuu `h2`-headerista ja `p`-tekstikappaleesta `div`-elementin sisällä:

```JS
import React from 'react';
import { styled } from '@mui/material/styles';

const CardWrapper = styled('div')(({ theme }) => ({
  padding: theme.spacing(3),
  backgroundColor: theme.palette.background.paper,

  '& .card-title': {
    color: theme.palette.primary.main,
    marginBottom: theme.spacing(2),
    fontSize: theme.typography.h5.fontSize,
  },

  '& .card-description': {
    color: theme.palette.text.secondary,
    marginBottom: theme.spacing(2),
    fontSize: theme.typography.body1.fontSize,
  },
}));

function CustomCard({ title, description }) {
  return (
    <CardWrapper>
      <h2 className="card-title">{title}</h2>
      <p className="card-description">{description}</p>
    </CardWrapper>
  );
}

export default CustomCard;
```

Esimerkissä luodaan `CustomCard`-komponentti, jonka otsake ja tekstikappale ottaavat tekstivärin, marginaalin ja fonttikoon teeman määrittelyistä. `Div`-elementti, jonka sisällä ne ovat, ottaa teemasta padding-määritykset sekä taustavärin. Jos tätä komponenttia käytetään eri teemojen sisällä, se mukautuu yleiseen ilmeeseen.

### FE-tehtävä
Suunnittele tilavaraussovellukselle uusi väripaletti ja valitse sille uusi fontti, ja tee niiden pohjalta sovellukselle uusi teema. Luo myös uusi taulukko- tai listakomponentti, jota käytät MUI-Tablen sijasta eri taulujen tietojen esittämiseen, ja toteuta se niin, että se käyttää MUI-teeman värejä ja fontteja.

### FE-Resursseja
- https://mui.com/material-ui/customization/theming/
- https://mui.com/material-ui/customization/creating-themed-components/
- https://www.dhiwise.com/post/exploring-mui-theming-options-customization-made-easy


### Backend

DRF muodostaa automaattisesti **selailtavan API:n,** jossa käyttäjä voi testailla endpointteja eri parametreilla. Tästä on hyötyä API:n kehityksessä ja testauksessa, ja myös FE-kehityksessä, kun FE-tiimi voi sen kautta helposti tarkistaa mitä parametreja ja tietotyyppejä endpointit ottavat vastaan.  

DRF:n selailtava API voi olla riittävä työkalu, mutta on olemassa myös API-standardi nimeltä [OpenAPI](https://swagger.io/resources/open-api/). OpenAPI on standardi, jonka perusteella REST API:lle voidaan tuottaa kieliagnostinen määrittely. Tätä määrittelyä voidaan käyttää viestittämään API:n ominaisuuksia eri toimijoiden kesken, ja myös automatisoida jotain sovelluskehityksen vaiheita, kuten juurikin luoda selailtavan API:n työkalulla kuten [Swagger](https://swagger.io/tools/open-source/). 

OpenAPIlla ja Swaggerilla saa siis tehtyä saman asian, mitä DRF tarjoaa sisäänrakennettuna, mutta niiden erona on se, että DRF:n oma selailtava API on DRF-spesifinen ratkaisu siinä missä OpenAPI on avoin standardi. Projektista riippuen tämä voi olla OpenAPI:n käyttöönoton vaatiman työn arvoista. 

DRF:lle on olemassa useita työkaluja, joilla sen API:sta saa automaattisesti generoitua OpenAPI-yhteensopivan dokumentaation ja Swagger-käyttöliittymän. Käytämme tässä `drf-yasg` –kirjastoa. 

`Drf-yasg` otetaan käyttöön komennolla `pip install drf-yasg`. Se konfiguroidaan `settings.py`:ssä seuraavasti:  

```Python
INSTALLED_APPS = [ 
   ... 
   'django.contrib.staticfiles',  
   'drf_yasg', 
   ... 
] 
```

Ja `urls.py`:ssä seuraavasti: 

```Python
from django.urls import re_path 
from rest_framework import permissions 
from drf_yasg.views import get_schema_view 
from drf_yasg import openapi 

schema_view = get_schema_view( 
   openapi.Info( 
      title="Tilavaraus API", 
      default_version='v1', 
      description="APIn kuvaus", 
      terms_of_service="https://www.google.com/policies/terms/", 
      contact=openapi.Contact(email="contact@tilavaraus.local"), 
      license=openapi.License(name="BSD License"), 
   ), 
   public=True, 
   permission_classes=(permissions.AllowAny,), 
)

urlpatterns = [ 
   path('swagger<format>/', schema_view.without_ui(cache_timeout=0), name='schema-json'), 
   path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'), 
] 
```

Tämä tarjoilee 3 polkua, joista API:n määrittelyjä voidaan tarkastella: 

- JSON -näkymä osoitteessa `/swagger.json `
- YAML -näkymä osoitteessa `/swagger.yaml` 
- swagger-ui -näkymä osoitteessa `/swagger/` 

OpenAPI käyttää määrittelyjen esittämiseen JSONin lisäksi [YAML](https://yaml.org/):ia. YAML on ihmisluettava datan serialisointiformaatti samaan tapaan kuin JSON, mutta se on JSONia helppolukuisempi, ja tavallaan JSONia ilman ylimääräisiä hipsuja ja sulkeita. YAMLia käytetään JSONin vaihtoehtona erilaisten konfiguraatio- ja määrittelytiedostojen kirjoittamiseen. Esimerkiksi seuraava JSON: 

```JSON
{ 
  "id": 0, 
  "nimi": "Kalle", 
  "ajokortti": true, 
  "osaaminen": [ 
    { 
      "kieli": "Python", 
      "taso": 3 
    }, 
    { 
      "kieli": "JavaScript", 
      "taso": 4 
    } 
  ] 
} 
```
Kääntyy YAML:ksi seuraavasti: 

```YAML
--- 
id: 0 
nimi: Kalle 
ajokortti: true 
osaaminen: 
- kieli: Python 
  taso: 3 
- kieli: JavaScript 
  taso: 4 
```

Kuten huomaat, YAML käyttää merkittävästi vähemmän välimerkkejä, ja esittää hierarkiaa ja rakennetta sisennyksillä sulkeiden sijaan, samaan tapaan kuin Python. 

### BE-tehtävä 

Ota `drf-yasg` käyttöön projektissa, ja tarkastele sen generoimia Swagger-näkymiä. 

### BE-Resursseja: 
- https://swagger.io/specification/
- https://drf-yasg.readthedocs.io/en/stable/
- https://yaml.org/
- https://www.bairesdev.com/tools/json2yaml/
