# CI/CD — Pipeline complet

Ce dossier documente l’ensemble du pipeline CI/CD utilisé pour :
- construire l’image Docker de l’application
- la pousser vers GitHub Container Registry (GHCR)
- déployer automatiquement sur un cluster k0s via kubectl
- exécuter toutes les étapes depuis un runner self‑hosted

Chaque étape est indépendante, testée et validée avant d’être intégrée au pipeline final.

---

## État actuel du pipeline

Le pipeline CI/CD est opérationnel et validé :

- Checkout : OK
- Build Docker : OK
- Push GHCR : OK
- Déploiement k0s : OK

Le runner self‑hosted exécute l’ensemble du workflow sans erreur.

---

## Structure de la documentation

```
docs/
└── ci-cd/
    ├── 00-intro.md
    ├── 01-checkout.md
    ├── 02-build.md
    ├── 03-push-ghcr.md
    ├── 04-deploy-k0s.md
    ├── glossaire.md
    └── README.md
```

---

## Architecture du pipeline

Le pipeline CI/CD complet suit les étapes suivantes :

1. Checkout du code  
2. Build Docker  
3. Push GHCR  
4. Déploiement k0s  

### Schéma simplifié du pipeline

```
+-------------------+
|    GitHub Repo    |
+-------------------+
          |
          v
+-------------------+
| Runner self-hosted|
+-------------------+
   |      |      |
   |      |      |
   v      v      v
Checkout  Build  Push GHCR
                 |
                 v
          +----------------+
          |      GHCR      |
          +----------------+
                 |
                 v
          +----------------+
          |   Cluster k0s  |
          +----------------+
                 |
                 v
            Application
```

---

## Workflow complet

```yaml
name: CI/CD

on:
  push:
    branches: ["main"]

jobs:
  build-and-deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout
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

## Secrets nécessaires

- GITHUB_TOKEN  
- KUBECONFIG  

---

## Comment tester

```
git commit --allow-empty -m "test ci-cd"
git push
```

---

## Troubleshooting

- permission denied lors du push GHCR  
- Cannot connect to the Docker daemon  
- certificate signed by unknown authority  
- connection refused  

---

## Conclusion

Pipeline automatisé, utilisant un runner self‑hosted, construisant et publiant une image Docker, puis déployant sur un cluster k0s.

