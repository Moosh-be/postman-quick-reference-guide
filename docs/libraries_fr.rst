************************
Bibliothèques JavaScript
************************

Postman est livré avec quelques bibliothèques intégrées.
Si vous préférez ajouter des bibliothèques JavaScript supplémentaires, veuillez consulter la section Bibliothèques personnalisées.

Bibliothèques JavaScript intégrées
----------------------------------

**cheerio**

Bibliothèque simple pour travailler avec le modèle DOM. Utile si vous récupérez du HTML. ::

    responseHTML = cheerio(pm.response.text());
    console.log(responseHTML.find('[name="firstName"]').val());

Exemple de récupération d'un code CSRF à partir de la balise meta: ::

    const $ = cheerio.load('<meta name="csrf" Content="the code">');
    console.log($("meta[name='csrf']").attr("content"));

Lire la suite: https://github.com/cheeriojs/cheerio

**crypto-js**

Bibliothèque qui implémente différentes fonctions cryptographiques.

Chaîne de hachage utilisant SHA256 ::

    CryptoJS.SHA256("some string").toString()

Chiffrement HMAC-SHA1 ::

    CryptoJS.HmacSHA1("Message", "Key").toString()

Chiffrement AES ::

    const encryptedText = CryptoJS.AES.encrypt('message', 'secret').toString();

Décryptage AES ::

    const plainText = CryptoJS.AES.decrypt(encryptedText, 'secret').toString(CryptoJS.enc.Utf8);

Lire la suite: https://www.npmjs.com/package/crypto-js

Bibliothèque Node.js
--------------------

Modules NodeJS disponibles dans Postman:

- path
- assert
- buffer
- util
- url
- punycode
- querystring
- string_decoder
- stream
- timers
- events


Bibliothèques personnalisées.
-----------------------------

Il n'existe aucun moyen standard d'inclure des bibliothèques JavaScript tierces.

Actuellement, le seul moyen est de récupérer (et éventuellement de stocker) le contenu de la bibliothèque JavaScript et d'utiliser la fonction JavaScript `eval` pour exécuter le code.

Modèle: ::

    pm.sendRequest("https://example.com/your-script.js", (error, response) => {
        if (error || response.code !== 200) {
            pm.expect.fail('Could not load external library');
        }

        eval(response.text());

        // VOTRE CODE ICI
    });

Exemple de chargement d'une bibliothèque en utilisant unpkg comme CDN: ::

    pm.sendRequest("https://unpkg.com/papaparse@5.1.0/papaparse.min.js", (error, response) => {
        if (error || response.code !== 200) {
            pm.expect.fail('Could not load external library');
        }

        eval(response.text());

        const csv = `id,name\n1,John`;
        const data = this.Papa.parse(csv); // notice the this
        console.log(data);
    });

Remarque: pour charger une bibliothèque et l'utiliser dans Postman, 
le code JavaScript doit être "compilé" et prêt pour la distribution.
Habituellement, le code sera disponible sous forme de fichier \*.min.js ou dans le dossier dist ou umd.
