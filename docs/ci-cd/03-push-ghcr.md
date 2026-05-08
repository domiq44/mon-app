# 03 — Push de l’image Docker vers GHCR

## 🎯 Objectif
Valider que le pipeline CI/CD est capable de :
- se connecter au registre GitHub Container Registry (GHCR)
- construire l’image Docker
- pousser l’image vers : `ghcr.io/domiq44/mon-app:latest`

Cette étape vérifie la partie “registry” du pipeline.

---

## 🧱 Pré‑requis

### 1. Étapes précédentes validées
- 01 — Checkout
- 02 — Build Docker (sans push)

### 2. Permissions GHCR
Deux options possibles :

### **Option A — Utiliser GITHUB_TOKEN (simple)**
Dans le dépôt GitHub :
- Settings → Actions → General → Workflow permissions  
  Activer :
  - ✔️ Read and write permissions  
  - ✔️ Allow GitHub Actions to publish packages  

Aucun secret supplémentaire n’est nécessaire.

### **Option B — Utiliser un token personnel GHCR_PAT (propre)**
Créer un token GitHub (Fine-grained PAT) avec :
- Packages → Read & Write
- Repository → Read

Puis ajouter dans :
Settings → Secrets → Actions :
- `GHCR_PAT`

---

## 🧩 Workflow minimal pour cette étape

Ce workflow :
1. Checkout
2. Login GHCR
3. Build + Push Docker

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
          username: domiq44
          password: \${{ secrets.GITHUB_TOKEN }}   # ou GHCR_PAT

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/domiq44/mon-app:latest
```

---

## 🔍 Comment tester

1. Faire un commit vide :
   ```
   git commit --allow-empty -m "test push ghcr"
   git push
   ```

2. Aller dans **Actions**, ouvrir le run.

3. Vérifier que l’étape **Build and push image** ne renvoie pas :
   - `permission_denied: write_package`
   - `denied: requested access to the resource is denied`
   - `unauthorized: authentication required`

---

## ✅ Comment valider que l’étape fonctionne

Dans les logs, tu dois voir :
- `#x exporting to image`
- `pushing layers`
- `pushed ghcr.io/domiq44/mon-app:latest`

Ensuite, vérifier sur GitHub :

1. Aller sur :  
   `https://github.com/domiq44?tab=packages`

2. Tu dois voir un package nommé :  
   **mon-app**

3. Cliquer dessus → vérifier :
   - la version `latest`
   - la date du push
   - la taille de l’image

---

## 📝 Notes importantes
- Si tu utilises `GITHUB_TOKEN`, les permissions doivent être activées dans les settings du dépôt.
- Si tu utilises `GHCR_PAT`, il doit être Fine-grained (les Classic PAT ne sont plus acceptés pour GHCR).
- Le push peut échouer si :
  - le runner n’a pas accès à Internet
  - GHCR est temporairement indisponible
  - les permissions ne sont pas activées

---

## 🔗 Étape suivante
Passer à **04 — Déploiement sur k0s avec kubectl**.

