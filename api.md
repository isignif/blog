---
layout: default
title: API
---

Vous êtes développeur et vous souaitez utiliser notre [API][api] pour automatiser la signification de vos demandes? Isignif met à disposition une [API RESTful][rest] qui vous permet de faire vos demandes sans quitter votre application.

Notre [API][api] est documentée au format [OpenAPI][openapi] et disponnible en ligne:

<p class="text-center">
    <a class="btn btn-primary btn-lg" href="https://github.com/isignif/openapi-definition">consulter la documentation OpenAPI</a>
</p>

Nous mettons à disposition un environnement de test disponnible à l'address suivantes: <https://test.isignif.fr>


## Exemple concret

Dans cet exemple nous allons faire le tour d'une demande de signification en utilisans l'API d'iSignif.

Nous utiliserons [cURL][curl] qui a l'avantage d'être présent sur la plupars des sytèmes d'exploitation.

### Obtenir un jeton d'authentification

La plupart des actions disponnibles via l'API de iSignif demande une **authentification**. Afin de pouvoir signifier, vous devez donc créer un compte et obtenir un **jeton d'authentification** qui nous permettra de retrouver votre utilisateur sur la plateforme.

Avant tout, vous avez besoin de créer votre profil. Nous avons deux types de comptes:

- le type `advocate` qui peut demander la signification d'acte
- le type `bailiff` qui peut demander la signification d'acte **ET** les signifier

Commençons donc par créer un compte de type `advocate`. En suivant la convention REST, vous pouvez deviner qu'il suffit de faire une requête de type _POST_ sur la ressource _advocate_. Vous devez fournir les paramètres nécessaires à la création d'un compte.

Voici un exemple avec le minimum d'informations nécessaire: 

~~~bash
$ curl -X POST \
       -d 'advocate[firstname]=Alexandre' \
       -d 'advocate[lastname]=Rousseau' \
       -d 'advocate[email]=isignif_api@dispostable.com' \
       -d 'advocate[password]=isignif_api' \
       -d 'advocate[password_confirmation]=isignif_api' \
       -d 'advocate[address_1]=2 rue des Lilas' \
       -d 'advocate[zip_code]=69001' \
       -d 'advocate[town]=Lyon' \
       https://test.isignif.fr/api/v1/advocates
~~~

A la suite de cette demande, vous allez recevoir un email de confirmation à l'adresse que vous avez rensignée.

<p class="alert alert-warning">Tant que l'addresse n'est pas confirmée, vous ne pouvez pas recevoir de jetons.</p>

Il suffit maintenant d'obtenir un jeton d'authentification en utilsant la ressource _tokens_:

~~~bash
$ curl -X POST \
       -d 'user[email]=isignif_api@dispostable.com' \
       -d 'user[password]=isignif_api' \
       https://test.isignif.fr/api/v1/tokens
~~~

Vous obtenez une reponse de cette forme

~~~json
{
  "token":"eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMjAsImVtYWlsIjoiaXNpZ25pZl9hcGlAZGlzcG9zdGFibGUuY29tIiwiZmlyc3RuYW1lIjoiQWxleGFuZHJlIiwibGFzdG5hbWUiOiJSb3Vzc2VhdSIsImV4cCI6MTU2OTg0NTk3MX0.rMqkGMN4Zr6WA7hTjxY8qUVPiNu2vYSLDP4k0EtpJ4A"
}
~~~

Vous pouvez remarquer que le token est un [JWT][jwt]. Vous pouvez donc le décoder très facilement le décoder (à l'aide d'une librairie comme [jwt-decode](https://github.com/auth0/jwt-decode) par exemple) et extraires ses infomrations:

~~~json
{
  "user_id": 120,
  "email": "isignif_api@dispostable.com",
  "firstname": "Alexandre",
  "lastname": "Rousseau",
  "exp": 1569845971
}
~~~

Vous remarquez certainement la clef `exp` qui contient le _timestamp_ de la date d'expiration du jeton. Elle est de 24h à partir de la date de création du jeton. Passé cette date, il faudra en générer un autre.

### Créer un acte

TODO


[api]: https://fr.wikipedia.org/wiki/Interface_de_programmation
[curl]: https://en.wikipedia.org/wiki/CURL
[rest]: https://fr.wikipedia.org/wiki/Representational_state_transfer
[openapi]: https://www.openapis.org/
[jwt]: https://jwt.io/