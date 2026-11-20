# TP7 – Notes de sécurité sur les actions tierces

## Observations sur `actions/setup-node`

- Badge **« créateur vérifié »** : présent (le dépôt appartient à l'organisation `actions`, l'organisation officielle GitHub).
- Nombre d'étoiles : **5 033** (relevé le 2026-11-25 via l'API GitHub).
- Code source : dossier **`src/`** (TypeScript). Le dossier `dist/` contient la version compilée réellement exécutée par l'action ; `action.yml` décrit l'action.

## Réponses

**Q3. Pourquoi faut-il épingler une version (`@v4`) plutôt que `@main` ?**

`@main` est une branche : son contenu peut changer à tout moment, sans que rien ne le signale dans notre workflow. Une action épinglée sur une version (`@v4`, et idéalement sur un SHA de commit complet) garantit que l'on exécute toujours le même code, de façon reproductible et auditable. Avec `@main`, une mise à jour malveillante ou une compromission du dépôt s'exécuterait immédiatement dans notre pipeline.

**Q4. Le badge « créateur vérifié » garantit-il qu'une action est sûre ? Justifiez.**

Non. Le badge certifie seulement que GitHub a vérifié l'identité de l'éditeur (propriété du domaine / de l'organisation). Il ne dit rien de la qualité ni de la sécurité du code : une action vérifiée peut contenir une vulnérabilité ou être compromise après coup. C'est un indice de confiance sur l'origine, pas une garantie de sûreté.

**Q5. Citez deux vérifications à faire avant d'utiliser, en production, une action d'un fournisseur inconnu.**

1. **Auditer le code** de l'action (source et `dist/`) et vérifier les permissions et secrets qu'elle demande, puis l'épingler sur un SHA de commit précis plutôt que sur une branche ou un tag mobile.
2. **Évaluer la réputation et la maintenance** : ancienneté, étoiles, releases, politique de sécurité, issues ouvertes, organisation éditrice ; et limiter les droits du `GITHUB_TOKEN` au strict nécessaire (`permissions:`), voire isoler l'action dans un job dédié.
