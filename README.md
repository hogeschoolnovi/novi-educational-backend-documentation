# NOVI Educational Backend

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
   * [Gebruiker aanpassen](#4-gebruiker-aanpassen)
   * [Alle gebruikers opvragen [admin]](#5-alle-gebruikers-opvragen-[admin])
   * [Beveiligd endpoint [user]](#6-beveiligd-endpoint-[user])
   * [Beveiligd endpoint [admin]](#7-beveiligd-endpoint-[admin])
* [Postman gebruiken](#rest-endpoint-benaderen-in-postman)


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
Alle rest-endpoints draaien op deze server: https://polar-lake-14365.herokuapp.com. Dit is de basis-uri. Alle voorbeeld-data betreffende de endpoints zijn in JSON format weergegeven. Wanneer er wordt vermeld dat er een token vereist is, betekent dit dat er een `Bearer` + `token` _header_ moet worden meegestuurd met het request:

```json
{
   "Content-Type": "application/json",
   "Authorization": "Bearer xxx.xxx.xxx",
}
```

### 0. Test
`GET /api/test/all`

Dit endpoint is vrij toegankelijk en is niet afgeschermd. Het is daarom een handig endpoint om te testen of het verbinden met de backend werkt. De response bevat een enkele string: `"Public Content."`

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

Het aanmaken van een nieuwe gebruiker (met admin-rol) vereist de volengende informatie:

```json
{
   "username": "klaas",
   "email" : "klaas@novi.nl",
   "password" : "123456",
   "role": ["admin"]
}
```

Het is ook mogelijk een gebruiker aan te maken met twee rollen:

```json
{
   "role": ["user", "admin"]
}
```

De response bevat een succesmelding.

_Let op_: er mogen géén additionele keys worden meegestuurd. Het e-mailadres moet altijd een e-mailadres zijn (met @ erin) anders geeft de backend een foutmelding.

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

Het opvragen van de gebruikersgegevens vereist een **token**. De response bevat alle informatie over de gebruiker zoals beschreven bij registratie.

### 4. Gebruiker aanpassen
`POST /api/user/update`

Het is mogelijk om een gebruiker zijn eigen e-mail of wachtwoord aan te laten passen. Dit vereist, naast de gegevens zelf, ook een **token**. Wanneer één van de twee entiteiten aangepast moeten worden, moet de andere data alsnog worden meegestuurd. Wachtwoorden moeten altijd een minimale lengte van 6 tekens hebben.

```
{
   "email" : "sjaak@sjaak.nl",
   "password": "123456",
   "repeatedPassword": "123456"
}
```

### 5. Alle gebruikers opvragen [admin]
`GET /api/admin/all`

Dit rest-endpoint geeft een lijst van alle gebruikers terug, maar is alleen toegankelijk voor gebruikers met de admin-rol. Het opvragen van deze gegevens vereist een **token**.

### 6. Beveiligd endpoint [user]
`GET /api/test/user`
Alleen gebruikers met een user-rol kunnen dit endpoint benaderen. Het opvragen van deze gegevens vereist een **token**. De response bevat een enkele string: `"User Content."`

### 7. Beveiligd endpoint [admin]
`GET /api/test/admin`
Alleen gebruikers met een admin-rol kunnen dit endpoint benaderen. Het opvragen van deze gegevens vereist een **token**. De response bevat een enkele string: `"Admin Board."` (IETS MEESTUREN?)

## Restpoints benaderen in Postman
Wanneer je een authorisation-token hebt ontvangen zal de backend bij alle beveiligde endpoints willen controleren wie de aanvrager is op basis van deze token. Dit zul je dus ook in Postman mee moeten geven.

![Plaatje postman met Authorization](img/auth_postman_example.png)

Onder het kopje headers voeg je als `Key` `Authorization` toe. Daarin zet je `<TOKEN TYPE> <SPATIE> <ACCESSTOKEN>` (zonder <>). 
