# 04 — Déploiement sur k0s avec kubectl

## 🎯 Objectif
Valider que le pipeline CI/CD est capable de :
- utiliser `kubectl` depuis le runner self‑hosted
- appliquer les manifests Kubernetes du dossier `k8s/`
- déployer l’application sur le cluster k0s
- utiliser un kubeconfig fourni via un secret GitHub

Cette étape valide la partie “CD” du pipeline.

---

## 🧱 Pré‑requis

### 1. Étapes précédentes validées
- 01 — Checkout
- 02 — Build Docker
- 03 — Push GHCR

### 2. Secret requis
Dans : Settings → Secrets → Actions  
- `KUBECONFIG`  
  Contenu complet du kubeconfig permettant d’accéder au cluster k0s.

### 3. Le runner doit avoir :
- `kubectl` installé (via l’action GitHub ou localement)
- accès réseau au cluster k0s

---

## 🧩 Workflow minimal pour cette étape

Ce workflow :
1. Checkout
2. Setup kubectl
3. Écrit le kubeconfig dans un fichier
4. Applique les manifests du dossier `k8s/`

```yaml
name: CI/CD - Step 04 Deploy k0s

on:
  push:
    branches: ["main"]

jobs:
  deploy-test:
    runs-on: self-hosted

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup kubectl
        uses: azure/setup-kubectl@v4

      - name: Write kubeconfig
        run: |
          echo "\${KUBECONFIG_CONTENT}" > kubeconfig
        env:
          KUBECONFIG_CONTENT: \${{ secrets.KUBECONFIG }}

      - name: Deploy to k0s
        run: kubectl apply --validate=false -f k8s/
        env:
          KUBECONFIG: \${{ github.workspace }}/kubeconfig
```

---

## 🔍 Comment tester

1. Faire un commit vide :
   ```
   git commit --allow-empty -m "test deploy k0s"
   git push
   ```

2. Aller dans **Actions**, ouvrir le run.

3. Vérifier que l’étape **Deploy to k0s** s’exécute sans erreur.

---

## ✅ Comment valider que l’étape fonctionne

Dans les logs, tu dois voir :
- `deployment.apps/mon-app configured`  
  ou  
- `deployment.apps/mon-app created`

Selon si c’est un premier déploiement ou une mise à jour.

Sur la machine du runner ou depuis n’importe quel poste ayant accès au cluster :

```
kubectl get pods -n default
```

Tu dois voir un pod du type :
```
mon-app-xxxxx   Running
```

Pour vérifier que la nouvelle image est bien utilisée :

```
kubectl describe deployment mon-app | grep Image
```

Tu dois voir :
```
Image: ghcr.io/domiq44/mon-app:latest
```

---

## 📝 Notes importantes
- Le kubeconfig doit être **complet**, incluant :
  - clusters
  - users
  - contexts
  - certificats
- Le runner doit pouvoir joindre l’API k0s (port 6443 par défaut).
- L’option `--validate=false` permet d’éviter les erreurs liées aux CRDs manquantes.
- Si `kubectl apply` échoue :
  - vérifier le kubeconfig
  - vérifier l’accès réseau
  - vérifier les droits RBAC

---

## 🔗 Étape suivante
Créer le **README global CI/CD** pour documenter l’ensemble du pipeline.

