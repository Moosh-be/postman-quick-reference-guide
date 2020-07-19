*****************
Schema validation
*****************

Cette section contient différents exemples de validation des réponses JSON à l'aide du validateur de schéma Ajv.
Je ne recommande pas d'utiliser le tv4 (Tiny Validator for JSON Schema v4).

La réponse est un objet
-----------------------

Voici la réponse JSON: ::

    {}


Voici le schéma JSON: ::

    const schema = {
        "type": "object",
    };

Et voici le test: ::

    pm.test("Valider le schema", () => {
        pm.response.to.have.jsonSchema(schema);
    });


L'objet a une propriété facultative
-----------------------------------

Voici la réponse JSON: ::

    {
        "code": "FX002"
    }

Il s'agit du schéma JSON avec une propriété nommée code de type String: ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" }
        }
    };


Les types possibles sont:
    - entier
    - nombre
    - booléen
    - null
    - Objet
    - tableau

L'objet a une propriété requise
-------------------------------

Compte tenu de cette réponse JSON: ::

    {
        "code": "FX002"
    }

Il s'agit du schéma JSON avec une propriété nommée "code" de type String qui est obligatoire: ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" }
        },
        "required": ["code"]
    };

Objets imbriqués
----------------

Compte tenu de cette réponse JSON: ::

    {
        "code": "2",
        "error": {
            "message": "Not permitted."
        }
    }

Il s'agit du schéma JSON avec un objet imbriqué nommé "error" qui a une propriété nommée "message" qui est une chaîne. ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" },
            "error": {
                "type": "object",
                "properties": {
                    "message": { type: "string" }
                },
                "required": ["message"]
            }
        },
        "required": ["code", "error"]
    };
