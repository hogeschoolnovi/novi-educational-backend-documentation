# NOVI Educational Backend

**Let op: deze backend wordt sinds 2023/11 niet meer gebruikt. De nieuwe backend en bijbehorende documentatie vindt je [hier](https://novi.datavortex.nl/).**

## Beschrijving
Deze backend is gebouwd door NOVI en mag alleen worden gebruikt voor opleidings-doeleinden.

Wanneer studenten de Frontend leerlijn volgen en een backend nodig hebben voor hun eindopdracht, kunnen zij ervoor kiezen om de NOVI backend te gebruiken. Deze backend ondersteunt alleen het registeren, inloggen en aanpassen van gebuikers. Het is niet mogelijk om andere informatie (naast `email`, `gebruikersnaam`, `wachtwoord` en `role`) op te slaan in deze database. _Let op_: de database met gebruikers wordt vaak binnen één uur weer geleegd.

De backend draait op een [Heroku](https://www.heroku.com/) server. Deze server wordt automatisch inactief wanneer er een tijdje geen requests gemaakt worden. De **eerste** request die de server weer uit de 'slaapstand' haalt zal daarom maximaal 30 seconden op zich kunnen laten wachten. Daarna zal de responsetijd normaal zijn. Voer daarom altijd eerst een test-request uit.

## Inhoud
* [Beschrijving](#beschrijving)
* [Gebruikersrollen](#gebruikersrollen)
* [Rest endpoints](#rest-endpoints)
   * [Testen](#0-test)
   * [Registreren](#1-registeren)
   * [Inloggen](#0-inloggen)
   * [Gebruiker opvragen](#3-gebruiker-opvragen)
   * [Profielfoto uploaden](#4-profielfoto-uploaden)
   * [Wachtwoord of email wijzigen](#5-)
   * [Alle gebruikers opvragen [admin]](#6-alle-gebruikers-opvragen-[admin])
   * [Beveiligd endpoint [user]](#7-beveiligd-endpoint-[user])
   * [Beveiligd endpoint [admin]](#8-beveiligd-endpoint-[admin])
   * [Errors](#9-errors)
* [Postman gebruiken](#rest-endpoint-benaderen-in-postman)
* [Update November 2021](#update-november-2021)

## Gebruikersrollen
Deze backend ondersteunt het gebruik van twee user-rollen:
1. `user`
2. `admin`

Elke gebruiker kan één of meerdere rollen hebben. Deze worden vastgesteld bij het _aanmaken_ van de gebruiker en kunnen daarna niet meer worden gewijzigd. Het is belangrijk om je te realiseren dat dat wanneer een gebruiker de `admin`-rol heeft dat deze dan niet _automatisch_ ook de `user`-rol heeft. 

Dat zit zo: stel, je maakt een gebruiker aan met een admin-rol en logt daarmee in. Als je met dit account REST-endpoints wil benaderen die toegankelijk zijn voor gebruikers met een `user` rol, zou je wellicht denken dat een admin dan automatisch ook geautoriseerd is om deze te bekijken. Dat is echter niet zo. Als je met een account dat alléén een admin rol heeft probeert om een user-endpoint te benaderen, geeft de applicatie de volgende error:

```
HTTP 401 Unauthorized
```

In de situatie dat een admin zowel gebruikers-rechten heeft als admin-rechten, krijgt deze dus twee rollen toegewezen. 

## Rest endpoints
Alle rest-endpoints draaien op deze server: https://frontend-educational-backend.herokuapp.com/. Dit is de basis-uri. Alle voorbeeld-data betreffende de endpoints zijn in JSON format weergegeven. Wanneer er wordt vermeld dat er een **token** vereist is, betekent dit dat er een `Bearer` + `token` _header_ moet worden meegestuurd met het request:

```json
headers: {
   "Content-Type": "application/json",
   "Authorization": "Bearer xxx.xxx.xxx",
}
```

### 0. Test
`GET /api/test/all`

Dit endpoint is vrij toegankelijk en is niet afgeschermd. Het is daarom een handig endpoint om te testen of het verbinden met de backend werkt. De response bevat een enkele string: `"De API is bereikbaar."`

### 1. Registreren
`POST /api/auth/signup`

Het aanmaken van een nieuwe gebruiker (met user-rol) vereist de volgende informatie:

```json
{
   "username": "piet",
   "email" : "piet@novi.nl",
   "password" : "123456",
   "role": ["user"]
}
```

Let hierbij op de volgende vereisten:
* Het emailadres moet daadwerkelijk een `@` bevatten
* Het wachtwoord en gebruikersnaam moeten **minimaal 6 tekens** bevatten
* Wanneer je een gebruiker probeert te registreren met een username die al bestaat, krijg je een foutcode. De details over deze foutmelding vindt je in `e.response`.

Indien de registratie succesvol was, ontvang je een succesmelding.

#### Optionele velden
Het is toegestaan om een _string_ mee te sturen onder de `info`-key, zodat je hier additionele informatie over de gebruiker in kunt opslaan:

```json
{
   "username": "piet",
   "email" : "piet@novi.nl",
   "password" : "123456",
   "info": "Ik woon in Utrecht",
   "role": ["user"]
}
```

#### Rollen
Wanneer je een gebruiker met admin-rol wil aanmaken, verander je de rol als volgt: "role": ["admin"]. Het is ook mogelijk een gebruiker aan te maken met twee rollen:

```json
{
   "role": ["user", "admin"]
}
```

### 2. Inloggen
`POST /api/auth/signin`

Het inloggen van een bestaande gebruiker kan alleen als deze al geregistreerd is. Inloggen vereist de volgende informatie:

```json
{
   "username": "user",
   "password" : "123456",
}
```

De response bevat een authorisatie-token (JWT) en alle gebruikersinformatie. Onderstaand voorbeeld laat de repsonse zien na het inloggen van een gebruiker met een admin-rol:

```json
{
    "id": 6,
    "username": "mod3",
    "email": "mod3@novi.nl",
    "roles": [
        "ROLE_USER",
        "ROLE_MODERATOR"
    ],
    "accessToken": "eyJhJIUzUxMiJ9.eyJzdWICJleQ0OTR9.AgP4vCsgw5TMj_AQAS-J8doHqADTA",
    "tokenType": "Bearer"
}
```

### 3. Gebruiker opvragen
`GET /api/user`

Het opvragen van de gebruikersgegevens vereist een **token**. Op die manier ziet de backend wiens gebruikersgegevens worden opgevraagd. Een gebruiker mag alleen zijn eigen gebruikersgegevens opvragen. De response bevat alle informatie over de gebruiker zoals beschreven bij registratie:

```json
{
    "id": 3,
    "username": "piet",
    "email": "piet@novi.nl",
    "info": "Ik woon in Utrecht",
    "roles": [
        "ROLE_USER"
    ]
}
```
### 4. Profielfoto uploaden
`POST /api/user/image`

Een gebruiker kan een profielfoto aan zijn profiel toevoegen. Dit vereist een **token**. De afbeelding moet worden aangeleverd in de vorm van een base64-string: 

```json
{
  "base64Image":            "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=="
}
```

Wanneer dit succesvol is, wordt het volledige gebruikers-object geretourneerd met alle huidige informatie over de gebruiker.

### 5. Gebruiker aanpassen
`PUT /api/user`

Het is mogelijk om een gebruiker zijn eigen e-mail, wachtwoord, profielfoto of informatie aan te laten passen. Dit vereist, naast de gegevens zelf, ook een token. Het aanpassen van het emailadres doe je als volgt:

```json
{
   "email" : "sjaak@sjaak.nl",
}
```

Het aanpassen van het wachtwoord doe je als volgt:

```json
{
   "password": "123456",
   "repeatedPassword": "123456"
}
```

Wanneer dit succesvol is, wordt het volledige gebruikers-object geretourneerd met alle huidige informatie over de gebruiker.

### 6. Alle gebruikers opvragen [admin]
`GET /api/admin/all`

Dit rest-endpoint geeft een lijst van alle gebruikers terug, maar is alleen toegankelijk voor gebruikers met de admin-rol. Het opvragen van deze gegevens vereist een **token**.

### 7. Beveiligd endpoint [user]
`GET /api/test/user`

Alleen gebruikers met een user-rol kunnen dit endpoint benaderen. Het opvragen van deze gegevens vereist een **token**. De response bevat een string.

### 8. Beveiligd endpoint [admin]
`GET /api/test/admin`
Alleen gebruikers met een admin-rol kunnen dit endpoint benaderen. Het opvragen van deze gegevens vereist een **token**. De response bevat een enkele string: `"Admin Board."`.

### 9. Errors
De backend kan verschillende errors teruggeven. We hebben ons best gedaan om deze allemaal af te vangen. Lees dan ook vooral de foutmelding in `e.response`.

## Restpoints benaderen in Postman
Wanneer je een authorisation-token hebt ontvangen zal de backend bij alle beveiligde endpoints willen controleren wie de aanvrager is op basis van deze token. Dit zul je dus ook in Postman mee moeten geven.

![Plaatje postman met Authorization](img/auth_postman_example.png)

Onder het kopje headers voeg je als `Key` `Authorization` toe. Daarin zet je `<TOKEN TYPE> <SPATIE> <ACCESSTOKEN>` (zonder <>). 

## Update November 2021
Deze backend is geüpdate en draait nu op een nieuw webadres en heeft meer functionaliteit. Maar geen zorgen, het oude adres: https://polar-lake-14365.herokuapp.com werkt nog steeds.
