*********
Workflows
*********

Comment extraire la valeur d'un jeton d'authentification à partir d'un corps de réponse de connexion et transmettre la demande suivante en tant que «jeton porteur» ?
---------------------------------------------------------------------------------------------------------------------------------------------------------------------


Compte tenu de la réponse du serveur d'authentification:

.. code-block:: javascript

    {
        "accessToken": "foo",
        "refreshToken": "bar"
        "expires": "1234"
    }

Extrayez la valeur du jeton de la réponse dans l'onglet **Tests**: ::

    var jsonData = pm.response.json();
    var token = jsonData.accessToken;

Définissez le jeton en tant que variable (globale, environnement, etc.) afin qu'il puisse être utilisé dans des requêtes ultérieures: ::

    pm.globals.set('token', token);

Pour utiliser le jeton dans la demande suivante, dans la partie en-têtes, les éléments suivants doivent être ajoutés (exemple de clé:valeur ci-dessous): ::
    Authorization:Bearer {‌{token}}


Comment lire les liens de la réponse et exécuter une demande pour chacun d'eux ?
--------------------------------------------------------------------------------

Avec la réponse: ::

    {
        "links": [
            "http://example.com/1",
            "http://example.com/2",
            "http://example.com/3",
            "http://example.com/4",
            "http://example.com/5"
        ]
    }

Avec le code suivant, nous lirons la réponse, parcourrons le tableau des liens et pour chaque lien, nous soumettrons une demande, en utilisant ``pm.sendRequest``. Pour chaque réponse, nous vérifions le code d'état. ::

    // Analyser la réponse
    var jsonData = pm.response.json();

    // Vérifier la réponse
    pm.test("La réponse contient des liens", function () {
        pm.response.to.have.status(200);
        pm.expect(jsonData.links).to.be.an('array').that.is.not.empty;
    });


    // Iterater la réponse
    var links = jsonData.links;

    links.forEach(function(link) {
        pm.test("Status code is 404", function () {
            pm.sendRequest(link, function (err, res) {
                pm.expect(res).to.have.property('code', 404);
            });
        });
    });



Comment créer un paramétrage de demande à partir d'un fichier Excel ou JSON ?
-----------------------------------------------------------------------------

TODO
