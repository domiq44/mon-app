# 01 — Checkout du code (Étape validée)

## 🎯 Objectif
Valider que le runner self‑hosted **feynman** est capable de :
- recevoir un job GitHub Actions
- cloner le dépôt `mon-app`
- préparer le workspace pour les étapes suivantes

Cette étape est la base de tout le pipeline CI/CD.

---

## 🧱 Pré‑requis
- Un runner self‑hosted en ligne dans : Settings → Actions → Runners
- Le workflow configuré avec :
  ```
  runs-on: self-hosted
  ```
- Aucun secret requis pour cette étape

---

## 🧩 Workflow minimal pour cette étape

Ce workflow ne fait que le checkout.  
Il permet de valider que le runner fonctionne correctement.

```yaml
name: CI/CD - Step 01 Checkout

on:
  push:
    branches: ["main"]

jobs:
  checkout-test:
    runs-on: self-hosted

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
```

---

## 🔍 Comment tester

1. Faire un commit vide :
   ```
   git commit --allow-empty -m "test checkout"
   git push
   ```

2. Aller dans **Actions**, sélectionner le workflow, ouvrir le run.

3. Vérifier que le job s’exécute sur :
   ```
   Runner: feynman (self-hosted)
   ```

---

## ✅ Comment valider que l’étape fonctionne

Dans les logs de l’étape **Checkout repository**, tu dois voir :
- `Cloning into '/home/.../_work/mon-app/mon-app'...`
- `Checked out commit XXXXXXX`
- Aucun message d’erreur Git (exit code 128, permissions, etc.)

Sur la machine du runner, tu peux vérifier que le dépôt a bien été cloné :

```
ls ~/github-runner/_work/mon-app/mon-app
```

Tu dois voir :
```
app.py
Dockerfile
k8s/
.github/
...
```

---

## 📝 Notes importantes
- Le runner ne doit jamais être installé dans le dépôt Git lui‑même.
- Le dossier `_work` est automatiquement géré par GitHub Actions.
- Cette étape doit être validée avant de passer au build Docker.

---

## 🔗 Étape suivante
Passer à **02 — Build de l’image Docker (sans push)**.


