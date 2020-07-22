******************
Postman Cheatsheet
******************

Merci d'avoir téléchargé cette aide-mémoire. Ce guide fait référence à l'application Postman, pas à l'extension Chrome. Veuillez signaler tout problème avec celui-ci.

Postman Cheat Sheet est basé sur la documentation officielle Postman et expérience propre.

Pour une documentation détaillée sur chaque fonctionnalité, consultez https://www.getpostman.com/docs.


Variables
=========

Toutes les variables peuvent être définies manuellement à l'aide de l'interface graphique de Postman et sont étendues.

Les extraits de code peuvent être utilisés pour travailler avec des variables dans des scripts (pré-demande, tests).

Pour en savoir plus sur les différentes portées de variables, consultez ce `tuto <https://medium.com/@vdespa/demystifying-postman-variables-how-and-when-to-use-different-variable-scopes-66ad8dc11200>`_.



Obtention de variables dans le générateur de demandes
-----------------------------------------------------

En fonction de la portée la plus proche:

Syntaxe: ``{{maVariable}}``

**Exemples**:

Request URL: ``http://{{domain}}/users/{{userId}}``

Headers (key:value): ``X-{{myHeaderName}}:foo``

Request body: ``{"id": "{{userId}}", "name": "John Doe"}``


Variables globales
------------------

Variables à usage général, idéales pour les résultats rapides et le prototypage.

Veuillez envisager d'utiliser l'une des variables plus spécifiques ci-dessous.
Supprimez les variables une fois qu'elles ne sont plus nécessaires.

*Quand utiliser :**

- transmission de données à d'autres requêtes

**Ecrire**

.. code-block:: javascript

    pm.globals.set('maVariable', MA_VALEUR);

**Lire** ::

    pm.globals.get('maVariable');

Alternativement, selon la portée: ::

    pm.variables.get('maVariable');

**Supprimer**

Supprimer une variable:

    pm.globals.unset('maVariable');

Supprimer TOUTES les variables globales (plutôt inhabituel) ::

    pm.globals.clear();

Variables de collection
-----------------------

**Quand les utiliser :**

- bonne alternative aux variables globales ou aux variables d'environnement
- pour les URL / informations d'authentification si un seul environnement existe

**Ecrire** ::

    pm.collectionVariables.set('maVariable', MA_VALEUR);

**Lire** ::

    pm.collectionVariables.get('maVariable');

**Supprimer** ::

    pm.collectionVariables.unset('maVariable');


Variables d'environment
-----------------------

Les variables d'environnement sont liées à l'environnement sélectionné.
Bonne alternative aux variables globales car elles ont une portée plus étroite.

** Quand utiliser: **

- stockage des informations spécifiques à l'environnement
- URL, informations d'authentification
- transmettre des données à d'autres demandes

**Ecrire** ::

    pm.environment.set('maVariable', MA_VALEUR);

**Lire** ::

    pm.environment.get('maVariable');

En fonction de la portée la plus proche: ::

    pm.variables.get('maVariable');

**Supprimer**

Supprimer une variable ::

    pm.environment.unset("maVariable");

Supprimer toutes les variables d'environment ::

    pm.environment.clear();

**Exemples**: ::

    pm.environment.set('name', 'John Doe');
    console.log(pm.environment.get('name'));
    console.log(pm.variables.get('name'));

** Détection du nom de l'environnement **

Si vous avez besoin de savoir à l'intérieur des scripts quel environnement est actuellement actif (locahost, production, ...) vous pouvez utiliser la propriété name: ::

    pm.environment.name


Variables de données
--------------------

Existe uniquement lors de l'exécution d'une itération (créée par Collection Runner ou Newman).

** Quand utiliser: **

- lorsque plusieurs ensembles de données sont nécessaires

**Ecrire**

Ne peut être défini qu'à partir d'un fichier CSV ou JSON.

**Lire** ::

    pm.iterationData.get('maVariable);

En fonction de la portée la plus proche: ::

    pm.variables.get('maVariable');

**Supprimer**

Ne peut être supprimé que du fichier CSV ou JSON.


Variables locales
-----------------

Les variables locales ne sont disponibles qu'avec la requête qui les a définies ou lors de l'utilisation de Newman / Collection runner pendant toute l'execution.

** Quand utiliser: **

- chaque fois que vous souhaitez remplacer toutes les autres portées de variables — pour une raison quelconque. Je ne suis pas sûr que cela soit nécessaire.

**Ecrire** ::

    pm.variables.set('maVariable', MA_VALEUR);

**Lire** ::

    pm.variables.get('maVariable', MA_VALEUR);

**Supprimer**

Les variables locales sont automatiquement supprimées une fois les tests exécutés.

Variables dynamiques
--------------------

Toutes les variables dynamiques peuvent être combinées avec des chaînes, afin de générer des données dynamiques / uniques.

Exemple de corps JSON:

.. code-block:: json

    {"name": "John Doe", "email": "john.doe.{{$timestamp}}@example.com"}

Si vous souhaitez utiliser des variables dynamiques dans les scripts, vous pouvez utiliser le `replaceIn` à partir de Postman v7.6.0. ::

    pm.variables.replaceIn('{{$randomFirstName}}'); // renvoie une chaîne

Pour plus de détails, veuillez consulter la section dédiée à :doc:`Dynamic variables </dynamic-variables>`

Variables de journalisation / débogage
--------------------------------------

Ouvrez Postman Console et utilisez `console.log` dans votre script de test ou de pré-requête.

Exemple: ::

    var myVar = pm.globals.get("myVar");
    console.log(myVar);

Assertions
==========

Remarque: vous devez ajouter l'une des assertions dans un callback ``pm.test``.

Exemple: ::

    pm.test("Le nom de votre test", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.value).to.eql(100);
    });

Status code
-----------

Vérifier si le code d'état est 200: ::

    pm.response.to.have.status(200);


Vérification de plusieurs codes d'état: ::

    pm.expect(pm.response.code).to.be.oneOf([201,202]);


Temps de résponse
-----------------

Temps de résponse inférieur à 100ms: ::

    pm.expect(pm.response.responseTime).to.be.below(9);

En-têtes
--------

L'en-tête existe: ::

    pm.response.to.have.header(X-Cache');

L'en-tête a une valeur: ::

    pm.expect(pm.response.headers.get('X-Cache')).to.eql('HIT');

Cookies
-------

Le cookie existe: ::

    pm.expect(pm.cookies.has('sessionId')).to.be.true;

Le cookie a une valeur: ::

    pm.expect(pm.cookies.get('sessionId')).to.eql(’ad3se3ss8sg7sg3');


Corps
-----

** Tout type de contenu / réponses HTML **

Correspondance exacte du corps: ::

    pm.response.to.have.body("OK");
    pm.response.to.have.body('{"success"=true}');

Correspondance partielle / le corps contient: ::

    pm.expect(pm.response.text()).to.include('Order placed.');

**Réponses JSON **

Analyser le corps (besoin de toutes les assertions): ::

    const response = pm.response.json();

Vérification de la valeur simple: ::

    pm.expect(response.age).to.eql(30);
    pm.expect(response.name).to.eql('John);

Vérification de la valeur imbriquée: ::

    pm.expect(response.products.0.category).to.eql('Detergent');

**Réponses XML**

Convertir le corps XML en JSON: ::

    const response = xml2Json(responseBody);

Remarque: consultez les assertions pour les réponses JSON.

Ignorer des tests
-----------------

Vous pouvez utiliser `pm.test.skip` pour sauter un test.
Les tests ignorés seront affichés dans les rapports.

** Exemple simple ** ::

    pm.test.skip("Status code is 200", () => {
        pm.response.to.have.status(200);
    });

** Saut conditionnel ** ::

    const shouldBeSkipped = true; // une condition

    (shouldBeSkipped ? pm.test.skip : pm.test)("Status code is 200", () => {
        pm.response.to.have.status(200);
    });

Provoquer l'échec d'un test
---------------------------

Vous pouvez faire échouer un test à partir des scripts sans écrire une assertion: ::

    pm.expect.fail('This failed because ...');

Bac à sable de Postman
======================

pm
---

il s'agit de l'objet contenant le script en cours d'exécution, qui peut accéder aux variables et a accès à une copie en lecture seule de la demande ou de la réponse.

pm.sendRequest
--------------

Permet d'envoyer **des requêtes GET simples en HTTP(S)** à partir de tests et de scripts de pré-requête.

Exemple: ::

    pm.sendRequest('https://httpbin.org/get', (error, response) => {
        if (error) throw new Error(error);
        console.log(response.json());
    });


Full-option **HTTP POST request with JSON body**: ::

    const payload = { name: 'John', age: 29};

    const options = {
        method: 'POST',
        url: 'https://httpbin.org/post',
        header: 'X-Foo:foo',
        body: {
            mode: 'raw',
            raw: JSON.stringify(payload)
        }
    };
    pm.sendRequest(options, (error, response) => {
        if (error) throw new Error(error);
        console.log(response.json());
    });

**Form-data POST request** (Postman will add the multipart/form-data header): ::

    const options = {
        'method': 'POST',
        'url': 'https://httpbin.org/post',
        'body': {
                'mode': 'formdata',
                'formdata': [
                    {'key':'foo', 'value':'bar'},
                    {'key':'bar', 'value':'foo'}
                ]
        }
    };
    pm.sendRequest(options, (error, response) => {
        if (error) throw new Error(error);
        console.log(response.json());
    });

** Envoi d'un fichier avec demande POST form-data **

Pour des raisons de sécurité, il n'est pas possible de télécharger un fichier à partir d'un script à l'aide de pm.sendRequest. Vous ne pouvez pas lire ou écrire des fichiers à partir de scripts.

Postman Echo
============

API Helper pour tester les demandes. Pour en savoir plus: https://docs.postman-echo.com.

** Obtenir l'heure UTC actuelle via le script de pré-requête ** ::

    pm.sendRequest('https://postman-echo.com/time/now', function (err, res) {
        if (err) { console.log(err); }
        else {
            var currentTime = res.stream.toString();
            console.log(currentTime);
            pm.environment.set("currentTime", currentTime);
        }
    });


Workflows
=========

Ne fonctionne qu'avec des exécutions de collection automatisées, comme avec Collection Runner ou Newman.
Cela n'aura aucun effet lors de l'utilisation dans l'application Postman.

De plus, il est important de noter que cela n'affectera que la prochaine requête en cours d'exécution.
Même si vous mettez ceci dans le script de pré-requête, il ne sautera PAS la requête en cours.


**Définir quelle sera la prochaine requête à exécuter**

``postman.setNextRequest(“Request name");``

**Arrêter d'exécuter les requêtes / arrêter l'exécution de la collecte**

``postman.setNextRequest(null);``
