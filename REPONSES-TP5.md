# TP5 – Construire un workflow CI avec GitHub Actions

## Réponses aux questions

### Étape 0 – Préparation

**Q0. Où se trouvent le code et les tests ? Quel fichier liste les dépendances de développement ? Existe-t-il un dossier `.github` ?**

- Le code applicatif est dans `app/` (`main.py`, `config.py`, `logging_config.py`).
- Les tests sont dans `tests/` (`test_config.py`, `test_endpoints.py`, `test_logging.py`).
- Les dépendances de développement sont listées dans `requirements-dev.txt` (pytest, pytest-cov, ruff, mypy, httpx).
- Oui, le dépôt possède un dossier `.github/`, avec `.github/workflows/` pour les workflows.

Vérification locale : `ruff check .` passe et `pytest` passe (18 tests, couverture 89,74 % ≥ 85 %).

### Étape 1 – Événements et filtres

**Q1. Compléter `01-filtres.yml`.**

```yaml
on:
  push:
    branches: [main, "feature/**"]
    paths: ["app/**", "tests/**"]
  pull_request:
    types: [opened, synchronize]
```

**Q2. Si on ne modifie que le README.md et qu'on pousse, le workflow se lance-t-il ? Et si on modifie un fichier de `app/` ?**

Non : le filtre `paths` exige qu'au moins un fichier sous `app/**` ou `tests/**` change. Un commit limité à `README.md` ne déclenche pas le workflow. En revanche, une modification dans `app/` (ou `tests/`) déclenche bien le workflow.

**Q3. Entre `branches` et `paths`, la logique est-elle ET ou OU ? Et entre deux types d'activité ?**

Entre `branches` et `paths`, c'est un **ET** : le push doit viser une branche autorisée **et** toucher un chemin autorisé. À l'intérieur d'une même liste (`branches`, `paths`), c'est un **OU**. Entre deux types d'activité (`opened`, `synchronize`), c'est aussi un **OU**.

**Q4. Créer `tag.yml`, déclenché uniquement par les tags `v*`.**

```yaml
on:
  push:
    tags: ["v*"]
```

Test : `git tag v1.0.0 && git push --tags` déclenche le workflow `Tag`.

### Étape 2 – Jobs, steps et needs

**Q5. Compléter les trois jobs.**

- `actions/checkout@v4` pour récupérer le code ;
- `actions/setup-python@v5` avec `python-version: "3.13"` ;
- `pip install -r requirements-dev.txt -r requirements.txt` ;
- `lint` et `build` tournent en parallèle, `test` dépend des deux : `needs: [lint, build]`.

**Q6. Que change le retrait de `needs` ? Et si on introduit une erreur de lint ?**

Sans `needs`, les trois jobs démarrent en parallèle : le graphe est plus large, la durée totale peut diminuer, mais `test` ne dépend plus de la réussite de `lint`/`build`. Avec `needs`, `test` attend la fin de `lint` et `build` ; si `lint` échoue, `test` est **skipped** et le run global est rouge.

**Q7. Pourquoi refaire checkout et setup-python dans le job `test` ?**

Parce que chaque job s'exécute sur un runner neuf et isolé : ni le code cloné ni l'environnement Python/dépendances ne persistent d'un job à l'autre. Chaque job doit donc reconstruire son propre environnement.

### Étape 3 – Contextes

**Q8. Quel contexte pour chaque valeur ?**

| Valeur | Contexte / expression |
|---|---|
| Nom de l'application | `env.APP_NAME` (ou `$APP_NAME`) |
| Acteur | `github.actor` |
| Nom de l'événement | `github.event_name` |
| Branche | `github.ref_name` |
| SHA du commit | `github.sha` |
| OS du runner | `runner.os` |

**Q9. Sur une PR vers `main`, comment changent `event_name`, `head_ref` et `base_ref` par rapport à un push ?**

Sur une PR : `github.event_name` vaut `pull_request`, `github.head_ref` contient la branche source (ex. `feature/x`) et `github.base_ref` contient la branche cible (`main`). Sur un push, `event_name` vaut `push` et `head_ref`/`base_ref` sont vides (la branche poussée est dans `github.ref_name`).

**Q10. Où retrouver le message du commit dans `toJson(github)` ?**

Dans le contexte d'un push, le message se trouve dans `github.event.head_commit.message` (visible dans le JSON sous `"head_commit": { "message": ... }`).
