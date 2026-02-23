# Déploiement du backend (Game_backend_deploy)

## Structure

- Ce dossier est une copie du backend GameForge prête pour le déploiement.
- La base de données utilisée est **MongoDB Atlas** (pas de conteneur MongoDB dans Docker).

## Étapes

### 1. Fichier `.env`

Créez ou éditez le fichier **`.env`** à la racine de ce dossier (`GameForgeAi-backend`), avec au minimum :

```env
MONGODB_URI=mongodb+srv://...   # URL fournie par MongoDB Atlas
JWT_SECRET=un-secret-long-32-caracteres-minimum
```

Vous pouvez vous inspirer de `.env.example`.

### 2. Lancer avec Docker

```bash
cd Game_backend_deploy/GameForgeAi-backend
docker compose up -d backend
```

### 3. Vérifier

- API : http://localhost:3000/api  
- Swagger : http://localhost:3000/api/docs  

### 4. Arrêter

```bash
docker compose down
```

## MongoDB Atlas

- Créez un cluster gratuit sur [cloud.mongodb.com](https://cloud.mongodb.com).
- **Network Access** : autorisez votre IP (ou `0.0.0.0/0` pour tester).
- **Database Access** : créez un utilisateur et récupérez l’URI (Connect → Drivers → Node.js).
