# CI/CD — Pipeline complet pour mon-app

Ce dossier documente l’ensemble du pipeline CI/CD utilisé pour :
- construire l’image Docker de l’application
- la pousser vers GitHub Container Registry (GHCR)
- déployer automatiquement sur un cluster k0s via kubectl
- exécuter toutes les étapes depuis un runner self‑hosted

Chaque étape est indépendante, testée et validée avant d’être intégrée au pipeline final.

---

## 📚 Structure de la documentation

```
docs/
└── ci-cd/
    ├── 01-checkout.md
    ├── 02-build.md
    ├── 03-push-ghcr.md
    ├── 04-deploy-k0s.md
    └── README.md   ← ce fichier
```

---

## 🧱 Architecture du pipeline

Le pipeline CI/CD complet suit les étapes suivantes :

1. **Checkout du code**  
   Le runner récupère le dépôt GitHub.

2. **Build Docker (local)**  
   Le runner construit l’image Docker localement.

3. **Push GHCR**  
   L’image est poussée vers :  
   `ghcr.io/domiq44/mon-app:latest`

4. **Déploiement k0s**  
   Les manifests du dossier `k8s/` sont appliqués sur le cluster.

---

## 🧩 Workflow complet

Voici le workflow final combinant toutes les étapes :

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
          username: domiq44
          password: \${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/domiq44/mon-app:latest

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

## 🔐 Secrets nécessaires

### 1. \`GITHUB_TOKEN\`
Doit avoir les permissions suivantes :
- Read & Write permissions
- Allow GitHub Actions to publish packages

### 2. \`KUBECONFIG\`
Contenu complet du kubeconfig permettant d’accéder au cluster k0s.

---

## 🧪 Comment tester le pipeline

1. Faire un commit vide :
   ```
   git commit --allow-empty -m "test ci-cd"
   git push
   ```

2. Aller dans GitHub → Actions  
3. Ouvrir le run  
4. Vérifier que chaque étape passe au vert

---

## 🩺 Troubleshooting

### ❌ Erreur : \`permission denied\` lors du push GHCR
- Vérifier les permissions du GITHUB_TOKEN
- Vérifier que le package existe dans GHCR

### ❌ Erreur : \`Cannot connect to the Docker daemon\`
- Vérifier que Docker tourne sur le runner
- Vérifier que l’utilisateur du runner appartient au groupe \`docker\`

### ❌ Erreur kubectl : \`certificate signed by unknown authority\`
- Vérifier que le kubeconfig contient bien les certificats CA

### ❌ Erreur kubectl : \`connection refused\`
- Vérifier que le runner peut joindre l’API k0s (port 6443)

---

## 🏁 Conclusion

Ce pipeline CI/CD :
- est entièrement automatisé
- utilise un runner self‑hosted
- construit et publie une image Docker
- déploie automatiquement sur un cluster k0s
- est documenté étape par étape pour faciliter la maintenance

Chaque fichier du dossier \`ci-cd\` détaille une étape précise du pipeline.


