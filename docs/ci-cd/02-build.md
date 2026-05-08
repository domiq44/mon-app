# 02 — Build de l’image Docker (sans push)

## Objectif
Tester la construction d’une image Docker sur un runner self‑hosted, sans la pousser vers un registre.

---

## Rappel : qu’est-ce qu’une image Docker ?

Une image Docker est un modèle utilisé pour créer des conteneurs.
Elle contient le code de l’application et tout ce qui est nécessaire pour l’exécuter.

---

## Pré‑requis

- Étape 01 validée
- Docker installé sur le runner
- L’utilisateur du runner doit pouvoir utiliser Docker

---

## Workflow minimal

```yaml
name: CI/CD - Step 02 Build

on:
  push:
    branches: ["main"]

jobs:
  build-test:
    runs-on: self-hosted

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build Docker image (no push)
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: mon-app:test
```

---

## Comment tester

```
git commit --allow-empty -m "test build"
git push
```

---

## Validation

Dans les logs :

- load build definition
- load metadata
- exporting to image

Sur le runner :

```
docker images | grep mon-app
```

---

## Erreurs fréquentes

- Cannot connect to the Docker daemon
- permission denied
- failed to solve

---

## Notes

- Cette étape ne pousse rien vers un registre.
