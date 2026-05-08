# 04 — Déploiement sur k0s avec kubectl

## Objectif
Déployer l’application sur un cluster k0s via kubectl.

---

## Rappel : qu’est-ce qu’un kubeconfig ?

Un kubeconfig est un fichier contenant les informations nécessaires pour que kubectl puisse se connecter à un cluster Kubernetes.

---

## Pré‑requis

- Étapes 01, 02 et 03 validées
- Secret KUBECONFIG présent dans GitHub
- kubectl installé sur le runner
- Accès réseau au cluster

---

## Workflow minimal

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
          echo "${KUBECONFIG_CONTENT}" > kubeconfig
        env:
          KUBECONFIG_CONTENT: ${{ secrets.KUBECONFIG }}

      - name: Deploy to k0s
        run: kubectl apply --validate=false -f k8s/
        env:
          KUBECONFIG: ${{ github.workspace }}/kubeconfig
```

---

## Comment tester

```
git commit --allow-empty -m "test deploy k0s"
git push
```

---

## Validation

Dans les logs :

- deployment.apps/<nom> created
- deployment.apps/<nom> configured

Sur le cluster :

```
kubectl get pods
```

---

## Erreurs fréquentes

- certificate signed by unknown authority
- connection refused
- erreurs RBAC

