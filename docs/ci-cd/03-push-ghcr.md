# 03 — Push de l’image Docker vers GHCR

## Objectif
Pousser une image Docker vers GitHub Container Registry (GHCR).

---

## Rappel : qu’est-ce qu’un registre d’images ?

Un registre est un serveur qui stocke des images Docker.
GHCR est le registre intégré à GitHub.

---

## Pré‑requis

- Étapes 01 et 02 validées
- Permissions GHCR configurées

### Option A — GITHUB_TOKEN
- Read and write permissions
- Allow GitHub Actions to publish packages

### Option B — GHCR_PAT
- Token personnel Fine-grained
- Permissions : Packages Read/Write, Repository Read

---

## Workflow minimal

```yaml
name: CI/CD - Step 03 Push GHCR

on:
  push:
    branches: ["main"]

jobs:
  push-test:
    runs-on: self-hosted

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: <votre-utilisateur-github>
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/<votre-utilisateur-github>/<nom-de-l-image>:latest
```

---

## Comment tester

```
git commit --allow-empty -m "test push ghcr"
git push
```

---

## Validation

Dans les logs :

- exporting to image
- pushing layers
- pushed ghcr.io/<utilisateur>/<image>:latest

---

## Erreurs fréquentes

- permission_denied: write_package
- unauthorized: authentication required
- denied: requested access to the resource is denied

