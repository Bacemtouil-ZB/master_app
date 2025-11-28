FROM node:22-alpine

WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code
COPY . .

# Exposer le port
EXPOSE 3000

# Lancer en mode développement
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "3000"]