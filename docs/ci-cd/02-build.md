
# 02 — Build de l’image Docker (sans push)

## 🎯 Objectif
Valider que le runner self‑hosted **feynman** est capable de :
- construire l’image Docker définie dans le `Dockerfile`
- utiliser Docker localement
- exécuter l’étape de build sans tenter de pousser l’image

Cette étape permet de vérifier que Docker fonctionne correctement sur le runner avant d’ajouter le push vers GHCR.

---

## 🧱 Pré‑requis
- Étape 01 (Checkout) validée
- Docker installé sur le runner self‑hosted
- Le runner doit appartenir au groupe permettant d’utiliser Docker (souvent `docker`)

---

## 🧩 Workflow minimal pour cette étape

Ce workflow effectue :
1. Le checkout du dépôt
2. Le build Docker **sans push**

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

## 🔍 Comment tester

1. Faire un commit vide :
   ```
   git commit --allow-empty -m "test build"
   git push
   ```

2. Aller dans **Actions**, ouvrir le run.

3. Vérifier que l’étape **Build Docker image (no push)** s’exécute correctement.

---

## ✅ Comment valider que l’étape fonctionne

Dans les logs de l’étape build, tu dois voir :
- `#1 [internal] load build definition`
- `#2 [internal] load .dockerignore`
- `#3 [internal] load metadata for ...`
- `#x exporting to image`
- Aucun message d’erreur du type :
  - `Cannot connect to the Docker daemon`
  - `permission denied`
  - `failed to solve`

Sur la machine du runner, tu peux vérifier que l’image a été construite :

```
docker images | grep mon-app
```

Tu dois voir une ligne similaire à :
```
mon-app   test   <image-id>   <date>
```

---

## 📝 Notes importantes
- Cette étape ne pousse rien vers GHCR.
- Elle permet d’isoler les problèmes Docker avant d’ajouter la partie registry.
- Si cette étape échoue, il faut vérifier :
  - que Docker tourne (`systemctl status docker`)
  - que l’utilisateur du runner peut utiliser Docker (`groups`)
  - que le Dockerfile est valide

---

## 🔗 Étape suivante
Passer à **03 — Push de l’image vers GHCR**.

