******
Newman
******

Comment définir un délai lors de l'exécution d'une collection ?
---------------------------------------------------------------

Vous avez une collection et avez l'obligation d'ajouter un délai de 10 secondes après chaque demande.

Pour ce faire, vous pouvez utiliser le paramètre ``--delay`` et spécifier un délai en millisecondes. ::

    newman run collection.json --delay 10000



Jenkins montre des caractères bizarres dans la console. Que faire ?
-------------------------------------------------------------------

.. image:: _static/jenkins-unicode.png
    :scale: 50 %

Si la sortie Newman de votre serveur CI ne s'affiche pas correctement, essayez d'ajouter les indicateurs suivants:
 ``--disable-unicode`` et / ou  ``--color off``

Exemple: ::

    newman run collection.json --disable-unicode



Comment passer dynamiquement le nom de la machine et le numéro de port lors de l'exécution des tests ?
------------------------------------------------------------------------------------------------------


Supposons que l'URL du serveur soumis au test puisse être différente chaque fois que vous obtenez un nouvel environnement de test,
ce qui est courant avec les environnements cloud.
C'est-à-dire que la partie nom_machine:numéro_port peut être différente.

Il peut y avoir plusieurs façons de le faire, voici donc une solution possible:

Vous pouvez définir des variables globales à l'aide de newman à partir de la ligne de commande. ::

    newman run my-collection.json --global-var "machineName=mymachine1234" --global-var "machinePort=8080"

Dans votre générateur de requêtes, utilisez-les simplement comme ``https://{machineName{}}:{‌{machinePort}}``.
