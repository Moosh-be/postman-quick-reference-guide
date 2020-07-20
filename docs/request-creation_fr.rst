*******************
Demande de création
*******************

J'ai une variable d'environnement comme {{url}}. Puis-je l'utiliser dans un script (comme pm.sendRequest) ?
-----------------------------------------------------------------------------------------------------------

La syntaxe suivante ne fonctionnera pas lors de l'écriture de scripts: ::

    pm.sendRequest({‌{url}}/mediaitem/)

Vous êtes à l'intérieur d'un script, vous devez donc utiliser l'API pm. * pour accéder à cette variable.
La syntaxe {‌{url}} ne fonctionne que dans le générateur de requêtes, pas dans les scripts.

Exemple: ::

    var requestUrl = pm.environment.get("url") + "/mediaitem/");

    pm.sendRequest(requestUrl, function (err, res) {
        // do stuff
    });


Comment utiliser le script de pre-request pour transmettre des données dynamiques dans le corps de la demande ?
---------------------------------------------------------------------------------------------------------------

Dans le script de pre-request, vous pouvez simplement créer un objet JavaScript,
définir les valeurs souhaitées et l'enregistrer en tant que variable ()

Par exemple, si vous souhaitez envoyer un corps de requête qui ressemble à: ::

    {
        "firstName": "Prénom",
        "lastName": "Nom",
        "email": "test@exemple.com"
    }

Vous pouvez effectuer les opérations suivantes dans le script de pré-requête: ::

    // Définir un nouvel objet
    var user = {
        "firstName": "Prénom",
        "lastName": "Nom",
        "email": "test@exemple.com"
    }

    // Enregistre l'objet en tant que variable.
    // JSON.stringify va sérialiser l'objet afin que Postman puisse l'enregistrer
    pm.globals.set('user', JSON.stringify(user));

Dans le corps de la demande, vous pouvez simplement utiliser ``{{user}} ``.
Cela fonctionne aussi bien pour les objets imbriqués: ::

    {
        "user": {{user}}
        "address": {
            "street": "Foo"
            "number": "2"
            "city": "Bar"
        }
    }

Comment puis-je modifier les en-têtes de demande ?
--------------------------------------------------

Vous pouvez modifier les en-têtes de demande à partir du script de pré-requête comme suit.

** Ajouter une en-tête ** ::

    pm.request.headers.add({
        key: 'X-Foo',
        value: 'Postman'
    });

** Supprimer l'en-tête ** ::

    pm.request.headers.remove('User-Agent'); // peut ne pas toujours fonctionner

** Mettre à jour l'en-tête ** ::

    pm.request.headers.upsert(
        {
            key: "User-Agent",
            value: "Not Postman"

        }
    );

Comment générer des données aléatoires ?
----------------------------------------

**Option 1** Utilisation des générateurs aléatoires Postman existants

Si vous devez créer une chaîne unique (avec chaque demande)
et la passer dans le corps de la demande,
dans l'exemple ci-dessous, une valeur GroupName unique sera généré à chaque exécution de la demande.

Vous pouvez utiliser la variable ``{{$guid}}`` - elle est automatiquement générée par Postman.
Ou vous pouvez utiliser l'horodatage actuel, ``{‌{$timestamp}}`` ::

    {
        "GroupName":"NomDeGroupe_{‌{$guid}}",
        "Description": "Exemple_API_Admin-Group_Description"
    }

Cela générera quelque chose comme: ::

    {
        "GroupName":"NomDeGroupe_0542bd53-f030-4e3b-b7bc-d496e71d16a0",
        "Description": "Exemple_API_Admin-Group_Description"
    }

L'inconvénient de cette méthode est que vous ne pouvez pas utiliser ces variables spéciales dans un script de pré-requête ou un test.
De plus, ils ne seront générés qu'une seule fois par requête, donc utiliser ``{‌{$guid}}`` plus d'une fois générera les mêmes données dans une requête.

**Option 2** Utilisation de générateurs aléatoires JavaScript existants

Ci-dessous, vous trouverez un exemple de fonction que vous pouvez utiliser pour générer un nombre entier dans un intervalle de valeurs spécifiques: ::

    function getRandomNumber(minValue, maxValue) {
        return Math.floor(Math.random() * (maxValue - minValue +1)) + minValue;
    }

Vous pouvez appeler la fonction comme ceci: ::

    var myRandomNumber = getRandomNumber(0, 100);

Et la sortie ressemblera à: ::

    67


Ci-dessous vous trouverez un exemple de fonction que vous pouvez utiliser pour générer des chaînes aléatoires: ::

    function getRandomString() {
        return Math.random().toString(36).substring(2);
    }

Vous pouvez appeler la fonction comme ceci: ::

    var myRandomNumber = getRandomString();

Et la sortie ressemblera à: ::

    5q04pes32yi


Comment déclencher une autre requête à partir du script de pré-requête ?
------------------------------------------------------------------------

**Option 1** Vous pouvez déclencher une autre requête dans la collection à partir du script de pré-requête en utilisant ``postman.setNextRequest``.

Cela peut être fait avec: ::

    postman.setNextRequest('Le nom de votre demande tel qu'il est enregistré dans Postman');

La difficulté est de revenir à la demande qui a initié l'appel. De plus, vous devez vous assurer de ne pas créer de boucles sans fin.

**Option 2** Une autre possibilité consiste à effectuer un appel HTTP à partir du script de pré-requête pour récupérer toutes les données dont vous pourriez avoir besoin.

Ci-dessous, je récupère un nom à partir d'une API distante et le définit comme une variable à utiliser dans la requête réelle qui s'exécutera juste après la fin du script de pré-requête: ::

    var options = { method: 'GET',
      url: 'http://www.mocky.io/v2/5a849eee300000580069b022'
    };

    pm.sendRequest(options, function (error, response) {
        if (error) throw new Error(error);
        var jsonData = response.json();
        pm.globals.set('name', 'jsonData.name');
    });

** Astuce ** Vous pouvez générer de telles demandes en utilisant le bouton générateur "Code" juste en dessous du bouton Enregistrer, une fois que vous avez une demande qui fonctionne.
Là, vous pouvez sélectionner NodeJS> Request et la syntaxe générée est très similaire à ce que Postman attend.

Vous pouvez importer cet exemple dans Postman en utilisant ce lien: https://www.getpostman.com/collections/5a61c265d4a7bbd8b303

Comment envoyer une requête avec un corps XML à partir d'un script ?
--------------------------------------------------------------------

Vous pouvez utiliser le modèle suivant pour envoyer une requête XML à partir d'un script.
Notez que `price` est une variable Postman qui sera remplacée. ::

    const xmlBody = `<?xml version="1.0"?>
    <catalog>
    <book id="bk101">
        <author>Gambardella, Matthew</author>
        <title>XML Developer's Guide</title>
        <genre>Computer</genre>
        <price>{{price}}</price>
        <publish_date>2000-10-01</publish_date>
        <description>An in-depth look at creating applications
        with XML.</description>
    </book>
    </catalog>`;

    const options = {
        'method': 'POST',
        'url': 'httpbin.org/post',
        'header': {
            'Content-Type': 'application/xml'
        },
        body: pm.variables.replaceIn(xmlBody) // replace any Postman variables
    }


    pm.sendRequest(options, function (error, response) {
        if (error) throw new Error(error);
        console.log(response.body);
    });

Comment passer des tableaux et des objets entre les requêtes ?
--------------------------------------------------------------

En supposant que votre réponse est au format JSON, vous pouvez extraire des données de la réponse en utilisant:

    var jsonData = pm.response.json();

Après cela, vous pouvez définir la réponse entière (ou juste un sous-ensemble comme celui-ci): ::

    pm.environment.set('myData', JSON.stringify(jsonData));

Vous devez utiliser JSON.stringify () avant d'enregistrer des objets / tableaux dans une variable Postman.
Sinon, cela pourrait ne pas fonctionner (selon votre version Postman ou Newman).

Dans la requête suivante où vous souhaitez récupérer les données, utilisez simplement:

- ``{myData}}`` si vous êtes dans le générateur de requêtes
- ``var myData = JSON.parse(pm.environment.get('myData'));``

L'utilisation des méthodes JSON.stringify et JSON.parse n'est pas nécessaire si les valeurs sont des chaînes, des entiers ou des booléens.

JSON.stringify () convertit une valeur en chaîne JSON tandis que la méthode JSON.parse () analyse une chaîne JSON, créant la valeur décrite par la chaîne.


Comment lire des fichiers externes ?
------------------------------------

Si certaines informations sont enregistrées dans un fichier localement sur votre ordinateur,
vous souhaiterez peut-être accéder à ces informations avec Postman.

Malheureusement, ce n'est pas vraiment possible.
Il existe un moyen de lire un fichier de données au format JSON ou CSV,
ce qui vous permet de rendre certaines variables dynamiques.
Ces variables sont appelées variables de données
et sont principalement utilisées pour tester différentes itérations sur une requête ou une collection spécifique.

Options possibles:

- démarrer un serveur local pour servir ce fichier et le récupérer dans Postman avec une requête GET.
- utilisez Newman comme script Node.js personnalisé et lisez le fichier à l'aide du système de fichiers.

Comment ajouter un délai entre les demandes de Postman ?
--------------------------------------------------------

Pour ajouter un délai après une requête, ajoutez ce qui suit dans vos tests: ::

    setTimeout(() => {}, 10000);

L'exemple ci-dessus ajoutera un délai de 10000 millisecondes ou 10 secondes.
