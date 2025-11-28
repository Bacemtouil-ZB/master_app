# 🚀 MERN Stack CI/CD Pipeline - Guide Complet et Professionnel

## 📋 Table des Matières

1. [Introduction](#introduction)
2. [Architecture du Projet](#architecture)
3. [Prérequis](#prérequis)
4. [Roadmap d'Implémentation](#roadmap)
5. [Configuration Docker](#docker)
6. [Configuration Jenkins](#jenkins)
7. [Pipeline CI/CD](#pipeline)
8. [Optimisations Avancées](#optimisations)
9. [Dépannage](#dépannage)
10. [Bonnes Pratiques](#bonnes-pratiques)

---

## 🎯 Introduction

### Qu'est-ce que ce projet ?

Ce projet est une **application MERN complète** (MongoDB, Express.js, React, Node.js) avec un pipeline **DevOps automatisé**. Il démontre l'intégration de technologies modernes pour créer un workflow de développement professionnel.

### Objectifs pédagogiques

- ✅ Comprendre la **containerisation** avec Docker
- ✅ Maîtriser l'**orchestration** avec Docker Compose
- ✅ Implémenter un **pipeline CI/CD** avec Jenkins
- ✅ Intégrer la **sécurité** avec Trivy
- ✅ Automatiser le **déploiement** vers Docker Hub
- ✅ Optimiser les **builds conditionnels**

### Technologies utilisées

| Technologie | Rôle | Version |
|-------------|------|---------|
| **Docker** | Containerisation | Latest |
| **Docker Compose** | Orchestration | v2.x |
| **Jenkins** | CI/CD Automation | LTS |
| **Trivy** | Security Scanning | Latest |
| **Docker Hub** | Image Registry | - |
| **MongoDB** | Base de données | Latest |
| **Express.js** | API Backend | v4.x |
| **React** | Frontend UI | v18.x |
| **Node.js** | Runtime | LTS Alpine |

---

## 🏗️ Architecture du Projet

### Structure des fichiers

```
master_app/
│
├── Client/                    # Application React (Frontend)
│   ├── src/                   # Code source React
│   ├── public/                # Fichiers statiques
│   ├── package.json           # Dépendances npm
│   └── Dockerfile             # Image Docker Client
│
├── Server/                    # API Node.js (Backend)
│   ├── routes/                # Routes Express
│   ├── models/                # Modèles MongoDB
│   ├── controllers/           # Logique métier
│   ├── package.json           # Dépendances npm
│   └── Dockerfile             # Image Docker Server
│
├── compose.yml                # Orchestration Docker Compose
├── Jenkinsfile                # Pipeline CI/CD Jenkins
└── README.md                  # Documentation (ce fichier)
```

### Flux de données

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │ ───> │   Server    │ ───> │   MongoDB   │
│  (React)    │ HTTP │  (Express)  │ DB   │  (NoSQL)    │
│  Port 3000  │      │  Port 9000  │      │  Port 27017 │
└─────────────┘      └─────────────┘      └─────────────┘
```

---

## 🔧 Prérequis

### Logiciels requis

| Logiciel | Installation | Vérification |
|----------|-------------|--------------|
| **Docker** | [docker.com](https://www.docker.com) | `docker --version` |
| **Docker Compose** | Inclus avec Docker Desktop | `docker compose version` |
| **Jenkins** | [jenkins.io](https://www.jenkins.io) | `java -jar jenkins.war` |
| **Git** | [git-scm.com](https://git-scm.com) | `git --version` |
| **Node.js** | [nodejs.org](https://nodejs.org) | `node --version` |

### Comptes nécessaires

- ☑️ Compte **Docker Hub** (gratuit) : [hub.docker.com](https://hub.docker.com)
- ☑️ Compte **GitHub** (pour le code source)
- ☑️ Accès **Jenkins** (local ou serveur)

### Connaissances recommandées

- 📘 Bases de **Docker** (images, conteneurs, volumes)
- 📗 Bases de **Git** (commit, push, pull)
- 📙 Bases de **JavaScript/Node.js**
- 📕 Notions de **CI/CD**

---

## 🗺️ Roadmap d'Implémentation

### Phase 1 : Containerisation

#### Étape 1.1 : Créer le Dockerfile Server

**Objectif** : Containeriser l'API Node.js/Express

**Fichier** : `Server/Dockerfile`

```dockerfile
# Image de base légère avec Node.js
FROM node:lts-alpine

# Définir le répertoire de travail dans le conteneur
WORKDIR /usr/src/app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier tout le code source
COPY . .

# Exposer le port de l'API
EXPOSE 9000

# Commande de démarrage
CMD ["npm", "start"]
```

**Explication ligne par ligne** :

- `FROM node:lts-alpine` : Utilise une image Node.js légère (Alpine Linux)
- `WORKDIR /usr/src/app` : Crée et définit le dossier de travail
- `COPY package*.json ./` : Copie package.json et package-lock.json
- `RUN npm install` : Installe les dépendances Node.js
- `COPY . .` : Copie tout le code source du serveur
- `EXPOSE 9000` : Déclare que le serveur écoute sur le port 9000
- `CMD ["npm", "start"]` : Lance l'application au démarrage du conteneur

**Tester localement** :

```bash
cd Server
docker build -t mern-server:test .
docker run -p 9000:9000 mern-server:test
```

---

#### Étape 1.2 : Créer le Dockerfile Client

**Objectif** : Containeriser l'application React avec un build multi-étapes

**Fichier** : `Client/Dockerfile`

```dockerfile
# ===== ÉTAPE 1 : BUILD =====
FROM node:lts-alpine AS build

WORKDIR /usr/src/app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances de développement
RUN npm install

# Copier le code source
COPY . .

# Créer le build de production React
RUN npm run build

# ===== ÉTAPE 2 : PRODUCTION =====
FROM node:lts-alpine

WORKDIR /usr/src/app

# Installer 'serve' pour servir les fichiers statiques
RUN npm install -g serve

# Copier uniquement le dossier build depuis l'étape précédente
COPY --from=build /usr/src/app/build ./build

# Exposer le port du client
EXPOSE 3000

# Servir l'application React
CMD ["serve", "-s", "build", "-l", "3000"]
```

**Pourquoi un build multi-étapes ?**

- ✅ **Optimisation** : L'image finale ne contient que les fichiers de production
- ✅ **Sécurité** : Pas de code source ni de dépendances dev dans l'image finale
- ✅ **Taille réduite** : Image plus légère et rapide à déployer

**Tester localement** :

```bash
cd Client
docker build -t mern-client:test .
docker run -p 3000:3000 mern-client:test
```

---

### Phase 2 : Orchestration 

#### Étape 2.1 : Créer le fichier Docker Compose

**Explication des concepts** :

| Concept | Explication |
|---------|-------------|
| **services** | Chaque service = un conteneur (mongo, server, client) |
| **ports** | Mapping port hôte:port conteneur (ex: 3000:3000) |
| **volumes** | Stockage persistant pour conserver les données MongoDB |
| **networks** | Réseau virtuel isolé pour la communication inter-conteneurs |
| **depends_on** | Définit l'ordre de démarrage des services |
| **environment** | Variables d'environnement passées au conteneur |

**Commandes essentielles** :

```bash
# Démarrer tous les services
docker compose up -d

# Voir les logs en temps réel
docker compose logs -f

# Arrêter tous les services
docker compose down

# Reconstruire les images
docker compose up --build

# Supprimer tout (conteneurs + volumes + réseaux)
docker compose down -v
```

---

### Phase 3 : Configuration Jenkins

#### Étape 3.1 : Installer les plugins Jenkins

**Objectif** : Ajouter les fonctionnalités Docker à Jenkins

**Procédure** :

1. Ouvrir Jenkins : `http://localhost:8080`
2. Aller dans **Manage Jenkins** > **Manage Plugins**
3. Onglet **Available**
4. Rechercher et installer :
   - ✅ **Docker Pipeline** (pour utiliser Docker dans les pipelines)
   - ✅ **Docker Commons** (dépendance automatique)
   - ✅ **Git Plugin** (pour cloner les repos)
5. Redémarrer Jenkins

**Vérification** :

```groovy
// Test dans un pipeline Jenkins
pipeline {
    agent any
    stages {
        stage('Test Docker') {
            steps {
                sh 'docker --version'
            }
        }
    }
}
```

---

#### Étape 3.2 : Configurer les credentials Docker Hub

**Objectif** : Permettre à Jenkins de pusher des images vers Docker Hub

**Procédure** :

1. **Manage Jenkins** > **Manage Credentials**
2. Cliquer sur **(global)** > **Add Credentials**
3. Remplir le formulaire :

| Champ | Valeur |
|-------|--------|
| **Kind** | Username with password |
| **Scope** | Global |
| **Username** | `bacemtouil` (votre username Docker Hub) |
| **Password** | Votre **Access Token** Docker Hub |
| **ID** | `3f2b5d95-2243-4892-ac19-7c8040616fcd` |
| **Description** | Docker Hub Credentials |

**Comment obtenir un Access Token Docker Hub ?**

1. Se connecter sur [hub.docker.com](https://hub.docker.com)
2. **Account Settings** > **Security** > **New Access Token**
3. Nom : `Jenkins Token`
4. Permissions : **Read & Write**
5. Copier le token (il ne sera visible qu'une fois)

---

#### Étape 3.3 : Créer le job Jenkins Pipeline

**Objectif** : Créer le pipeline qui exécutera le CI/CD

**Procédure** :

1. **New Item** > Nom : `MERN-Pipeline` > Type : **Pipeline**
2. Dans **Pipeline** :
   - **Definition** : `Pipeline script from SCM`
   - **SCM** : `Git`
   - **Repository URL** : `https://github.com/votre-username/master_app`
   - **Branch** : `*/main` (ou `*/master`)
   - **Script Path** : `Jenkinsfile`
3. Dans **Build Triggers** :
   - ☑️ Cocher **Poll SCM**
   - Schedule : `H/5 * * * *` (vérifie toutes les 5 minutes)

**Explication du Schedule** :

```
H/5 * * * *
│   │ │ │ │
│   │ │ │ └─── Jour de la semaine (0-7, dimanche=0 ou 7)
│   │ │ └───── Mois (1-12)
│   │ └─────── Jour du mois (1-31)
│   └───────── Heure (0-23)
└───────────── Minute (H = hash pour répartir la charge)
```

---

### Phase 4 : Pipeline CI/CD 

**Explication détaillée de chaque étape** :

| Étape | Objectif | Commande |
|-------|----------|----------|
| **Checkout Code** | Clone le code depuis Git | `checkout scm` |
| **Build Server Image** | Construit l'image Docker du serveur | `docker.build()` |
| **Build Client Image** | Construit l'image Docker du client | `docker.build()` |
| **Security Scan** | Analyse les vulnérabilités avec Trivy | `docker run trivy` |
| **Push Images** | Envoie les images vers Docker Hub | `docker push` |
| **Post Actions** | Nettoie les ressources Docker | `docker system prune` |

---


**Avantages des builds conditionnels** :

- ⚡ **Performance** : Build 50% plus rapide en moyenne
- 💰 **Coûts réduits** : Moins de ressources CPU/RAM utilisées
- 🔄 **Flexibilité** : Adapte le pipeline au contexte
- ✅ **Qualité** : Même niveau de tests et sécurité

---

## 📊 Vérification et Tests

### Test 1 : Vérifier les images localement

```bash
# Lister toutes les images Docker
docker images

# Vous devriez voir :
# bacemtouil/mern-server    latest    ...
# bacemtouil/mern-client    latest    ...
```

### Test 2 : Vérifier sur Docker Hub

1. Aller sur [hub.docker.com](https://hub.docker.com)
2. Se connecter avec votre compte
3. Vérifier que vos repositories contiennent les images :
   - `bacemtouil/mern-server`
   - `bacemtouil/mern-client`

### Test 3 : Tester le pipeline Jenkins

```bash
# 1. Faire un changement dans le code
echo "// Test" >> Server/index.js

# 2. Commit et push
git add .
git commit -m "test: trigger pipeline"
git push origin main

# 3. Attendre 5 minutes (pollSCM)
# 4. Vérifier dans Jenkins que le pipeline s'est déclenché
```

---

## 🐛 Dépannage

### Problème 1 : Jenkins ne détecte pas les changements

**Solution** :

```bash
# Vérifier la configuration Git dans Jenkins
# Manage Jenkins > Configure System > Git plugin
# Vérifier que le polling fonctionne :
cat /var/jenkins_home/logs/tasks/SCM\ polling.log
```

### Problème 2 : Échec du push vers Docker Hub

**Causes possibles** :

- ❌ Credentials incorrects
- ❌ Token Docker Hub expiré
- ❌ Repository n'existe pas sur Docker Hub

**Solution** :

```bash
# Tester manuellement la connexion Docker Hub
docker login -u bacemtouil
# Entrer le token

# Vérifier les credentials dans Jenkins
# Manage Jenkins > Manage Credentials
```

### Problème 3 : Trivy ne trouve pas l'image

**Solution** :

```bash
# Vérifier que l'image est bien buildée
docker images | grep mern

# Tester Trivy manuellement
docker run --rm \
-v /var/run/docker.sock:/var/run/docker.sock \
aquasec/trivy:latest image bacemtouil/mern-server:latest
```

---

## ✅ Bonnes Pratiques

### Sécurité

- 🔒 **Ne jamais** commiter de secrets dans Git
- 🔑 Utiliser des **tokens** au lieu de mots de passe
- 🛡️ Scanner les images avec **Trivy** avant déploiement
- 🔐 Limiter les **permissions** des credentials Jenkins

### Performance

- ⚡ Utiliser le **build cache** Docker (`--cache-from`)
- 🎯 Builds **conditionnels** pour gagner du temps
- 🧹 **Nettoyer** régulièrement les images inutilisées
- 📦 Images **Alpine** pour réduire la taille

### Maintenance

- 📝 **Documenter** chaque changement important
- 🏷️ **Versionner** les images (`v1.0.0`, `v1.1.0`, etc.)
- 📊 **Monitorer** les logs Jenkins et Docker
- 🔄 **Mettre à jour** régulièrement les dépendances

---

## 🎓 Conclusion

### Ce que vous avez appris

- ✅ **Containeriser** une application MERN complète
- ✅ **Orchestrer** plusieurs services avec Docker Compose
- ✅ **Automatiser** le CI/CD avec Jenkins Pipeline
- ✅ **Sécuriser** les images avec Trivy
- ✅ **Optimiser** les builds avec des conditions
- ✅ **Déployer** vers un registry (Docker Hub)

### Prochaines étapes

- 🚀 **Kubernetes** : Déployer sur un cluster K8s
- 🌐 **Production** : Configurer un reverse proxy (Nginx)
- 📊 **Monitoring** : Ajouter Prometheus + Grafana
- 🧪 **Tests** : Intégrer des tests unitaires et E2E
- 🔄 **GitOps** : Mettre en place ArgoCD ou Flux

### Ressources utiles

- 📚 [Documentation Docker](https://docs.docker.com)
- 📘 [Documentation Jenkins](https://www.jenkins.io/doc/)
- 📙 [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- 📗 [Best Practices Docker](https://docs.docker.com/develop/dev-best-practices/)

---

