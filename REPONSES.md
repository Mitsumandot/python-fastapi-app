# TP6 – Variables d'environnement, Inputs et Outputs

## Réponses aux questions

**Q1. Pourquoi la variable n'est-elle pas disponible dans la step qui l'a définie, mais seulement dans les suivantes ?**

`GITHUB_ENV` pointe vers un fichier que le runner lit **à la fin** de la step en cours. Les lignes écrites sont ensuite injectées comme variables d'environnement au début des steps suivantes du même job. La step qui écrit ne voit donc pas encore la variable qu'elle vient d'ajouter.

**Q2. Ajoutez un second job `verify-env` qui affiche `$APP_NAME` : que constate-t-on ? Quelle est la portée de `GITHUB_ENV` ?**

La variable est vide dans `verify-env` : `GITHUB_ENV` est limité au **job** dans lequel elle est écrite. Chaque job tourne sur un runner séparé et repart d'un environnement vierge.

**Q3. Que se passe-t-il si on écrit `>` au lieu de `>>` dans `GITHUB_OUTPUT` alors que deux outputs sont définis dans la même step ?**

`>` écrase le fichier à chaque écriture : seul le **dernier** output écrit subsiste, le premier est perdu (et l'output correspondant sera vide). Il faut `>>` pour ajouter chaque ligne au fichier.

**Q4. Pourquoi `GITHUB_ENV` ne suffit-il pas ici, alors que `GITHUB_OUTPUT` fonctionne entre jobs ?**

`GITHUB_ENV` ne partage des variables qu'à l'intérieur d'un même job. Pour passer une valeur à un autre job, il faut déclarer un `outputs:` au niveau du job, alimenté par `steps.<id>.outputs.<nom>` (écrit dans `GITHUB_OUTPUT`), puis le lire via `needs.<job>.outputs.<nom>` dans le job dépendant.

**Q5. Que se passe-t-il si on retire `needs: build` du job `deploy` ?**

`deploy` n'attend plus `build` : il peut démarrer en parallèle, et `needs.build.outputs.full_version` ne référence plus de job déclaré, donc l'expression est vide. Le déploiement affiche une version vide et n'a plus aucune garantie que `build` a réussi.
