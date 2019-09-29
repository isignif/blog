---
layout: default
title: API
---

Vous êtes développeur et vous souaitez utiliser notre [API][api] pour automatiser la signification de vos demandes? Isignif met à disposition une [API RESTful][rest] qui vous permet de faire vos demandes sans quitter votre application.

Notre [API][api] est documentée au format [OpenAPI][openapi] et disponnible en ligne:

<p class="text-center">
    <a class="btn btn-primary btn-lg" href="https://github.com/isignif/openapi-definition">consulter la documentation OpenAPI</a>
</p>

Nous mettons à disposition un environnement de test disponible à l'address suivantes: <https://test.isignif.fr>

## Sommaire

* TOC
{:toc}

## Normes

Afin de rendre cette API utilisable, nous avons respecté deux principales normes:

- [API RESTful][rest] qui spécifie comment accéder et comment **interagir** avec des ressources
- [JSON:API][jsonapi] qui définie l'orga

## Workflow d'un acte

1. Création d'un acte
2. Paramétrage & ajout de significations
3. Confirmation de l'acte
4. 

## Exemple d'une demande de création d'un acte

Dans cet exemple nous allons faire le tour d'une demande de signification en utilisant l'API d'iSignif.

Nous utiliserons [cURL][curl] qui a l'avantage d'être présent sur la plupart des systèmes d'exploitation.

### Obtenir un jeton d'authentification

La plupart des actions disponibles via l'API de iSignif demande une **authentification**. Afin de pouvoir signifier, vous devez donc créer un compte et obtenir un **jeton d'authentification** qui nous permettra de retrouver votre utilisateur sur la plateforme.

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

A la suite de cette demande, vous allez recevoir un email de confirmation à l'adresse que vous avez renseignée.

<p class="alert alert-warning">Tant que l'addresse n'est pas confirmée, vous ne pouvez pas recevoir de jetons.</p>

Il suffit maintenant d'obtenir un jeton d'authentification en utilisant la ressource _tokens_:

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

### Utiliser le token

Vous pouvez remarquer que le token est un [JWT][jwt]. Vous pouvez donc le décoder très facilement le décoder (à l'aide d'une librairie comme [jwt-decode](https://github.com/auth0/jwt-decode) par exemple) et extraire ses informations:

~~~json
{
    "user_id": 120,
    "email": "isignif_api@dispostable.com",
    "firstname": "Alexandre",
    "lastname": "Rousseau",
    "exp": 1569845971
}
~~~


<p class="alert alert-info">Vous remarquez certainement la clef `exp` qui contient le _timestamp_ de la date d'expiration du jeton. Elle est de 24h à partir de la date de création du jeton. Passé cette date, il faudra en générer un autre.</p>

Utilisons donc le `user_id` récupéré dans le JWT pour consulter nos informations:

~~~bash
$ curl -H "Authorization: eyJhb...k0EtpJ4A" https://test.isignif.fr/api/v1/advocates/120
~~~

~~~json
{
    "data": {
        "id": "120",
        "type": "advocate",
        "attributes": {
            "email": "isignif_api@dispostable.com",
            "firstname": "Alexandre",
            "lastname": "Rousseau",
            "activated": true,
            // ...
        },
        "relationships": {
            "acts": {
                "data": []
            }
        },
        "links": {
            "self": "https://test.isignif.fr/advocates/120.json"
        }
    },
    "included": []
}
~~~

Et voilà. 

<p class="alert alert-info">Si vous êtes fin connaisseur, vous reconnaissez sûrement la syntaxe <a class="alert-link" href="https://jsonapi.org/">JSON:API</a>. Cette syntaxe permet de structurer le format du JSON.</p>

### Créer un acte

Maintenant que nous avons un jeton d'authentification nous pouvons effectuer une demande de signification. Cette demande est un **acte**.

Un acte est lié à un **type d'acte** qui définit sont prix. Utilisons donc l'API afin de trouver quel type d'acte nous voulons signifier. Vous pouvez utiliser le paramètre `term` pour filtrer les résultats.

Voici un exemple en cherchant "nantissement":

~~~bash
$ curl http://localhost:3000/api/v1/act_types?term=nantissement
~~~
~~~json
{
    "data": [
        {
            "id": "196",
            "type": "act_type",
            "attributes": {
                "name": "Signification à la société du nantissement des parts sociales"
            },
        },
        // ...
    ]
}
~~~

Maintenant que nous avons un `act_type_id`, nous pouvons créer un acte.

~~~bash
$ curl -X POST \
       -H "Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMjAsImVtYWlsIjoiaXNpZ25pZl9hcGlAZGlzcG9zdGFibGUuY29tIiwiZmlyc3RuYW1lIjoiQWxleGFuZHJlIiwibGFzdG5hbWUiOiJSb3Vzc2VhdSIsImV4cCI6MTU2OTg0NTk3MX0.rMqkGMN4Zr6WA7hTjxY8qUVPiNu2vYSLDP4k0EtpJ4A" \
       -d 'act[act_type_id]=196' \
       https://test.isignif.fr/api/v1/acts
~~~

~~~json
{
    "data": {
        "id": "99",
        "type": "act",
        "attributes": {
            "advocate_id": 120,
            "act_type_id": 196,
            "current_step": "created",
            // ...
        },
        "relationships": {
            // ...
        },
        "links": {
            "self": "https://test.isignif.fr/acts/99.json"
        }
    },
    "included": []
}
~~~

Et voilà! Mais ce n'est pas terminé. Il nous reste deux choses à faire:

1. ajouter des significations sur cet acte
2. le confirmer

### Ajouter des significations

Une signification est attachée à **une ville** que nous devons chercher:

~~~bash
$ curl -X POST -d 'term=villejuif'  https://test.isignif.fr/api/v1/towns/search
~~~

~~~json
[
    {
        "text":"Villejuif (94800)",
        "id":10435
    }
]
~~~

Nous pouvons donc noter ce `town_id` et créer la signification:

~~~bash
$ curl -X POST \
       -H "Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMjAsImVtYWlsIjoiaXNpZ25pZl9hcGlAZGlzcG9zdGFibGUuY29tIiwiZmlyc3RuYW1lIjoiQWxleGFuZHJlIiwibGFzdG5hbWUiOiJSb3Vzc2VhdSIsImV4cCI6MTU2OTg0NTk3MX0.rMqkGMN4Zr6WA7hTjxY8qUVPiNu2vYSLDP4k0EtpJ4A" \
       -d 'signification[name]=Chez mémé' \
       -d 'signification[town_id]=10435' \
       https://test.isignif.fr/api/v1/acts/99/significations
~~~

Vous pouvez consulter toutes les significations crées sur l'acte avec cette requête:

~~~bash
$ curl -H "Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMjAsImVtYWlsIjoiaXNpZ25pZl9hcGlAZGlzcG9zdGFibGUuY29tIiwiZmlyc3RuYW1lIjoiQWxleGFuZHJlIiwibGFzdG5hbWUiOiJSb3Vzc2VhdSIsImV4cCI6MTU2OTg0NTk3MX0.rMqkGMN4Zr6WA7hTjxY8qUVPiNu2vYSLDP4k0EtpJ4A" \
       https://test.isignif.fr/api/v1/acts/99
~~~

~~~json
{
    "data": {
        "id": "99",
        "type": "act",
        // ...
    },
    "included": [
        {
            "id": "196",
            "type": "act_type",
            "attributes": {
                "name": "Signification aux créanciers de l’acte de nantissement de l’outillage et du matériel d’équipement"
            },
            // ...
        },
        {
            "id": "132",
            "type": "signification",
            "attributes": {
                "name": "Chez mémé",
                "created_at": "2019-09-29T19:59:52.000+02:00",
                "updated_at": "2019-09-29T19:59:52.000+02:00",
                "bailiff_id": 36,
                "act_id": 99,
                "town_id": 10435,
                "bailiff_comment": null
            },
            // ...
        }
    ]
}
~~~

### Confirmer notre acte

Il est maintenant temps de confirmer notre acte. Cette action aura pour conséquence d'engager l'acte et de contacter toutes les huissiers sélectionnés pour les significations.

~~~bash
$ curl -X POST \
       -H "Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMjAsImVtYWlsIjoiaXNpZ25pZl9hcGlAZGlzcG9zdGFibGUuY29tIiwiZmlyc3RuYW1lIjoiQWxleGFuZHJlIiwibGFzdG5hbWUiOiJSb3Vzc2VhdSIsImV4cCI6MTU2OTg0NTk3MX0.rMqkGMN4Zr6WA7hTjxY8qUVPiNu2vYSLDP4k0EtpJ4A" \
       https://test.isignif.fr/api/v1/acts/99/confirm
~~~

Vous pouvez constater que le statut de l'acte vient de changer en consultant l'historique de l'acte:

~~~bash
$ curl -H "Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMjAsImVtYWlsIjoiaXNpZ25pZl9hcGlAZGlzcG9zdGFibGUuY29tIiwiZmlyc3RuYW1lIjoiQWxleGFuZHJlIiwibGFzdG5hbWUiOiJSb3Vzc2VhdSIsImV4cCI6MTU2OTg0NTk3MX0.rMqkGMN4Zr6WA7hTjxY8qUVPiNu2vYSLDP4k0EtpJ4A" \
       https://test.isignif.fr/api/v1/acts/99/act_histories
~~~

~~~json
{
    "data": [
        {
            "id": "178",
            "type": "act_history",
            "attributes": {
                "step": "created",
                "user_id": 120,
                "act_id": 99,
                "signification_id": null,
                "created_at": "2019-09-29T19:29:35.000+02:00",
                "updated_at": "2019-09-29T19:29:35.000+02:00"
            },
            // ...
        },
        {
            "id": "180",
            "type": "act_history",
            "attributes": {
                "step": "confirmed",
                "user_id": 120,
                "act_id": 99,
                "signification_id": null,
                "created_at": "2019-09-29T20:14:06.000+02:00",
                "updated_at": "2019-09-29T20:14:06.000+02:00"
            },
            // ...
        }
    ],
    "included": [
        // ...
    ]
}
~~~

TODO

## Exemple de gestion de signification d'un acte

TODO

[api]: https://fr.wikipedia.org/wiki/Interface_de_programmation
[curl]: https://en.wikipedia.org/wiki/CURL
[rest]: https://fr.wikipedia.org/wiki/Representational_state_transfer
[openapi]: https://www.openapis.org/
[jsonapi]: https://jsonapi.org/
[jwt]: https://jwt.io/