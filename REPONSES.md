# TP8 – Stratégies GitHub Actions

## Réponses aux questions

**Q1. Combien de jobs sont générés ? Donnez la liste des combinaisons.**

La matrice croise 2 systèmes d'exploitation et 3 versions de Python : `2 × 3 = 6` jobs.

- `ubuntu-latest` + Python 3.11
- `ubuntu-latest` + Python 3.12
- `ubuntu-latest` + Python 3.13
- `windows-latest` + Python 3.11
- `windows-latest` + Python 3.12
- `windows-latest` + Python 3.13

**Q2. Combien de jobs au total ? Rappel : exclude s'applique avant include.**

Toujours **6 jobs** : la matrice de base en produit 6, `exclude` en retire 1 (`windows-latest` + 3.11), puis `include` en ajoute 1 (`ubuntu-latest` + 3.14, marqué `experimental: true`) → 6 − 1 + 1 = 6.

- `ubuntu-latest` + 3.11
- `ubuntu-latest` + 3.12
- `ubuntu-latest` + 3.13
- `ubuntu-latest` + 3.14 (`experimental: true`)
- `windows-latest` + 3.12
- `windows-latest` + 3.13

**Q3. Que se passerait-il si vous tentiez d'exclure la combinaison 3.14 ? Pourquoi ?**

L'exclusion serait sans effet : `exclude` ne s'applique qu'aux combinaisons issues du produit cartésien de base (`os` × `python-version`). La combinaison 3.14 n'existe pas dans cette base, elle est ajoutée ensuite par `include`. Comme `exclude` est évalué avant `include`, aucune entrée ne correspond et le job `ubuntu-latest` + 3.14 continue de tourner.

**Q4. Quelle est la différence observée ? Dans quel cas choisiriez-vous true ? false ?**

Avec `fail-fast: true` (valeur par défaut), dès qu'un job de la matrice échoue, GitHub annule les jobs encore en cours ou en attente : le run s'arrête au premier échec, les jobs non terminés passent en `cancelled`. Avec `fail-fast: false`, tous les jobs de la matrice vont jusqu'au bout : on obtient le résultat complet de chaque combinaison, y compris les autres échecs.

Observation sur ce dépôt : avec `fail-fast: true`, 4 jobs réussissent (Ubuntu 3.11/3.12/3.13 et 3.14 expérimental), `windows-latest` + 3.13 échoue et `windows-latest` + 3.12 est annulé. Avec `fail-fast: false`, les 6 jobs vont au bout : 4 succès et 2 échecs (les deux jobs Windows).

On choisit `true` quand on veut un retour rapide sur un build manifestement cassé et économiser des minutes de runner. On choisit `false` quand on a besoin de la vision complète de la matrice (bug spécifique à une plateforme ou à une version de Python, tests instables) ou quand on veut corriger tous les échecs en une seule passe.

**Q5. Donnez deux raisons d'utiliser max-parallel en entreprise.**

1. **Limiter la charge sur des ressources partagées** : un déploiement, une base de données de test, une API externe ou une licence ne supportent qu'un nombre limité d'accès simultanés ; `max-parallel` évite de les saturer ou de se faire limiter (rate limiting).
2. **Maîtriser le coût et la consommation de runners** : sur des runners payants ou un quota d'organisation, limiter le parallélisme évite de monopoliser tous les runners disponibles et laisse des capacités aux autres workflows.
