# TP7 – Actions tierces et actions personnalisées

## Réponses aux questions

**Q1. Pourquoi `actions/checkout` n'a-t-il pas besoin de `with` alors que `setup-node` en a besoin ?**

`checkout` a des valeurs par défaut pertinentes : par défaut il récupère le dépôt et le commit de l'événement qui a déclenché le workflow. `setup-node`, lui, doit savoir **quelle version de Node.js** installer : il n'existe pas de version par défaut universelle, elle dépend du projet. D'où le paramètre obligatoire `node-version`.

**Q2. Pourquoi le job `verify` ne retrouve-t-il pas `report.txt` sans l'artefact, alors que les deux jobs partagent le même dépôt ?**

Les deux jobs ne partagent pas de système de fichiers : chacun tourne sur un runner neuf et fait son propre checkout du dépôt au même commit. `report.txt` est produit pendant l'exécution de `build`, il n'est pas dans le dépôt Git. Sans artefact, il n'existe donc pas sur le runner de `verify`.

**Q6. Pourquoi `actions/checkout` doit-il s'exécuter avant l'appel d'une action locale ?**

Une action locale (`uses: ./.github/actions/build-info`) est lue depuis le workspace du runner. Sans `checkout`, le dossier `.github/actions/build-info` n'existe pas sur le runner et l'action est introuvable.

**Q7. Pourquoi `shell: bash` est-il obligatoire sur les steps `run` d'une action composite ?**

Contrairement aux steps d'un workflow classique, une action composite n'a pas de shell par défaut : elle peut s'exécuter sur Linux, macOS ou Windows, et GitHub ne peut pas deviner le shell voulu. Il faut donc le déclarer explicitement (`shell: bash`, `shell: pwsh`, etc.) sur chaque step `run`.
