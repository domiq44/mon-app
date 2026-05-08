# 01 — Checkout du code

## Objectif
Récupérer automatiquement le code du dépôt GitHub sur un runner self‑hosted.

---

## Rappel : qu’est-ce qu’un runner self-hosted ?

Un runner self-hosted est une machine (PC, VM, serveur) que vous contrôlez et sur laquelle GitHub Actions exécute les workflows.

---

## Pré‑requis

- Un runner self‑hosted installé et en ligne.
- Le workflow doit utiliser : runs-on: self-hosted

---

## Workflow minimal

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

## Comment tester

1. Faire un commit vide :

```
git commit --allow-empty -m "test checkout"
git push
```

2. Aller dans Actions.
3. Vérifier que le job s’exécute sur le runner self-hosted.

---

## Validation

Dans les logs :

- Cloning into '/.../_work/...'
- Checked out commit XXXXXXX

Sur la machine du runner :

```
ls <chemin-du-runner>/_work/<nom-du-repo>/<nom-du-repo>
```

---

## Erreurs fréquentes

- Runner offline
- Workflow exécuté sur ubuntu-latest au lieu de self-hosted
- Permission denied si le runner est installé dans un dossier Git

---

## Notes

- Le dossier _work est créé automatiquement.
- Le runner ne doit jamais être installé dans un dépôt Git.

