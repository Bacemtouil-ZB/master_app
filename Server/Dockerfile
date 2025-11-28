# Utiliser l'image de base Node.js (version légère) — utiliser une version alpine spécifique patchée
FROM node:20-alpine
# Définir le répertoire de travail dans le conteneur
WORKDIR /app

# Copier les fichiers package.json et package-lock.json
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier tout le code source du serveur
COPY . .

# Exposer le port 9000
EXPOSE 9000

# Commande pour démarrer l'application
CMD ["npm", "start"]