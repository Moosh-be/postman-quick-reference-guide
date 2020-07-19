**********
Assertions
**********

Les assertions dans Postman sont basée sur les fonctionnalités de la librairie Chai Assertion.
Vous pouvez lire une documentation complète de Chai en visitant http://chaijs.com/api/bdd/

Comment trouver un objet dans un tableau par la valeur de la propertié ?
------------------------------------------------------------------------

Sur base de la réponse suivante: ::

    {
        "companyId": 10101,
        "regionId": 36554,
        "filters": [
            {
                "id": 101,
                "name": "VENDOR",
                "isAllowed": false
            },
            {
                "id": 102,
                "name": "COUNTRY",
                "isAllowed": true
            },
            {
                "id": 103,
                "name": "MANUFACTURER",
                "isAllowed": false
            }
        ]
    }

Vérifier que la propriété isAllowed est vraie pour le filtre COUNTRY. ::

    pm.test("Vérifier que le filtre country est permis", function () {
        // Parser le corps de response
        var jsonData = pm.response.json();

        // Trouver l'index du tableau array pour COUNTRY
        var countryFilterIndex = jsonData.filters.map(
                function(filter) {
                    return filter.name; // <-- ICI on a le nom de la propriété
                }
            ).indexOf('COUNTRY'); // <-- ICI On a la valeur qu'on cherche

        // Get the country filter object by using the index calculated above
        var countryFilter = jsonData.filters[countryFilterIndex];

        // Check that the country filter exists
        pm.expect(countryFilter).to.exist;

        // Check that the country filter is allowed
        pm.expect(countryFilter.isAllowed).to.be.true;
    });


How find nested object by object name?
--------------------------------------

Sur base de la réponse suivante: ::

    {
        "id": "5a866bd667ff15546b84ab78",
        "limits": {
            "59974328d59230f9a3f946fe": {
                "lists": {
                    "openPerBoard": {
                        "count": 13,
                        "status": "ok", <-- CHECK ME
                        "disableAt": 950,
                        "warnAt": 900
                    },
                    "totalPerBoard": {
                        "count": 20,
                        "status": "ok",  <-- CHECK ME
                        "disableAt": 950,
                        "warnAt": 900
                    }
                }
            }
        }
    }

You want to check the value of the `status` in both objects (openPerBoard, totalPerBoard). The problem is that in order to reach both objects you need first to reach the lists object, which itself is a property of a randomly named object (59974328d59230f9a3f946fe).

So we could write the whole path ``limits.59974328d59230f9a3f946fe.lists.openPerBoard.status`` but this will probably work only once.

For that reason it is first needed to search inside the ``limits`` object for the ``lists`` object. In order to make the code more readable, we will create a function for that: ::

    function findObjectContaininsLists(limits) {
        // Iterate over the properties (keys) in the object
        for (var key in limits) {
            // console.log(key, limits[key]);
            // If the property is lists, return the lists object
            if (limits[key].hasOwnProperty('lists')) {
                // console.log(limits[key].lists);
                return limits[key].lists;
            }
        }
    }

The function will iterate over the limits array to see if any object contains a ``lists`` object.

Next all we need to do is to call the function and the assertions will be trivial: ::

    pm.test("Vérifier le statut", function () {
        // Parse JSON body
        var jsonData = pm.response.json();

        // Retrieve the lists object
        var lists = findObjectContaininsLists(jsonData.limits);
        pm.expect(lists.openPerBoard.status).to.eql('ok');
        pm.expect(lists.totalPerBoard.status).to.eql('ok');
    });


Comment comparer la valeur d'une réponse et une variable déjà défine ?
----------------------------------------------------------------------

Lets presume you have a value from a previous response (or other source) that is saved to a variable. ::

    // Récuperer les valeurs de la réponse
    var jsonData = pm.response.json();
    var username = jsonData.userName;

    // Saving the value for later use
    pm.globals.set("username", username);

How do you compare that variable with values from another API response?

In order to access the variable in the script, you need to use a special method, basically the companion of setting a variable. Curly brackets will not work in this case: ::

    pm.test("Your test name", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.value).to.eql(pm.globals.get("username"));
    });

Comment comparer la valeur d'une réponse parmi plusieurs valeurs valides ?
--------------------------------------------------------------------------

Sur base de la réponse suivante: ::

    {
        "SiteId": "aaa-ccc-xxx",
        "ACL": [
            {
                "GroupId": "xxx-xxx-xx-xxx-xx",
                "TargetType": "Subscriber"
            }
        ]
    }

Vous voulez vérifier que ``TargetType`` est *Subscriber* ou *Customer*.

L'assertion peut ressembler à ceci: ::

    pm.test("Doit être subscriber ou customer", function () {
        var jsonData = pm.response.json();
        pm.expect(.TargetType).to.be.oneOf(["Subscriber", "Customer"]);
    });

où:
    - jsonData.ACL[0] est le premier élément du tableau d'ACL
    - to.be.oneOf permet un tableau des valeurs valides possibles


Comment analyser une réponse HTML pour extraire une valeur spécifique ?
-----------------------------------------------------------------------

Supposons que vous souhaitiez obtenir la valeur de champ masqué _csrf pour les assertions ou une utilisation ultérieure à partir de la réponse ci-dessous: ::

    <form name="loginForm" action="/login" method="POST">
            <input type="hidden" name="_csrf" value="a0e2d230-9d3e-4066-97ce-f1c3cdc37915" />
            <ul>
                <li>
                    <label for="username">Username:</label>
                    <input required type="text" id="username" name="username" />
                </li>
                <li>
                    <label for="password">Password:</label>
                    <input required type="password" id="password" name="password" />
                </li>
                <li>
                    <input name="submit" type="submit" value="Login" />
                </li>
            </ul>
    </form>

Pour analyser et récupérer la valeur, nous utiliserons la bibliothèque JavaScript cheerio: ::

    // Parse HTML and get the CSRF token
    responseHTML = cheerio(pm.response.text());
    console.log(responseHTML.find('[name="_csrf"]').val());

Cheerio est conçu pour une utilisation sans navigateur et implémente un sous-ensemble de la fonctionnalité jQuery. En savoir plus à ce sujet sur https://github.com/cheeriojs/cheerio


Comment réparer l'erreur "ReferenceError: jsonData is not defined" ?
--------------------------------------------------------------------

Si vous avez un script comme celui-ci: ::

    pm.test("Le nom doit être John", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.name).to.eql('John');
    });

    pm.globals.set('name', jsonData.name);


Vous devriez avoir l'erreur ``ReferenceError: jsonData is not defined`` en définissant la variable globale.

La raison est que ``jsonData`` est uniquement défini à l'intérieur de la portée de la fonction anonyme (la partie avec ``function() {...} `` à l'intérieur de ``pm.test``).  Vous essayez de définir les variables globales qui se trouvent en dehors de la fonction, donc ``jsonData`` n'est pas défini. ``jsonData`` ne peut exister que dans l'étendue où il a été défini.

Vous avez donc plusieurs façons de gérer cela:

1. définissez  ``jsonData`` en dehors de la fonction, par exemple avant votre fonction pm.test (préférée) ::

    var jsonData = pm.response.json(); <-- callback externe défini

    pm.test("Le nom doit être John", function () {
        pm.expect(jsonData.name).to.eql('John');
    });

    pm.globals.set('name', jsonData.name);


2. Définissez l'environnement ou la variable globale à l'intérieur de la fonction anonyme (j'éviterais personnellement de mélanger les tests / assertions avec les variables de réglage, mais cela fonctionnerait). ::

    pm.test("Le nom doit être John", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.name).to.eql('John');
        pm.globals.set('name', jsonData.name); // <-- utilisation dans le callback
    });

J'espère que cela aide et clarifie un peu l'erreur.

Comment faire une assertion de correspondance d'objet partielle ?
-----------------------------------------------------------------

Avec la réponse: ::

    {
        "uid": "12344",
        "pid": "8896",
        "firstName": "Jane",
        "lastName": "Doe",
        "companyName": "ACME"
    }

Vous voulez valider qu'une partie de la réponse a une valeur spécifique.
Par exemple, vous n'êtes pas intéressé par la valeur dynamique de uid et pid
mais vous voulez valider celles de firstName, lastName et companyName.

Vous pouvez faire une correspondance partielle de la réponse en utilisant l'expression ``to.include``.
Vous pouvez éventuellement vérifier l'existence des propriétés supplémentaires sans vérifier la valeur. ::

    pm.test("Doit inclure un objet", function () {
        var jsonData = pm.response.json();
        var expectedObject = {
            "firstName": "Jane",
            "lastName": "Doe",
            "companyName": "ACME"
        }
        pm.expect(jsonData).to.include(expectedObject);

        // Optional check if properties actually exist
        pm.expect(jsonData).to.have.property('uid');
        pm.expect(jsonData).to.have.property('pid');
    });
