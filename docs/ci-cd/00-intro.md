# 00 — Introduction au pipeline CI/CD

## Objectif
Ce document présente, en termes simples, ce que fait le pipeline CI/CD et comment les différentes étapes s’enchaînent.

---

## Qu’est-ce qu’un pipeline CI/CD ?

Un pipeline CI/CD est une suite d’étapes automatisées qui s’exécutent à chaque changement de code.
Dans ce projet, il permet de :

- récupérer le code depuis GitHub
- construire une image Docker
- publier cette image dans un registre (GHCR)
- déployer l’application sur un cluster k0s

---

## Vue d’ensemble

Schéma simplifié du flux :

```
Code → Build → Image → Registry → Cluster → Application
```

Plus en détail :

1. Le code est poussé sur GitHub.
2. Un runner self-hosted exécute un workflow GitHub Actions.
3. Le runner construit une image Docker à partir du code.
4. L’image est poussée dans GitHub Container Registry (GHCR).
5. Le cluster k0s récupère cette image et déploie l’application.

---

## Les composants principaux

- GitHub : héberge le code et les workflows CI/CD.
- Runner self-hosted : machine qui exécute les workflows.
- Docker : construit les images à partir du code.
- GHCR : registre où sont stockées les images.
- k0s : distribution légère de Kubernetes utilisée comme cluster.
- kubectl : outil en ligne de commande pour parler au cluster.

---

## Comment lire la documentation

Les fichiers sont organisés par étape :

- 01 — Checkout : récupérer le code
- 02 — Build : construire l’image Docker
- 03 — Push GHCR : publier l’image
- 04 — Deploy k0s : déployer sur le cluster

Le fichier README.md résume l’ensemble du pipeline.  
Le fichier glossaire.md définit les termes techniques utilisés.

