# Déployer le front GameForge (Flutter Web) sur Render

Projet : **Game_front_deploy/GameForgeAi-fronts** (Flutter Web, fork/clone du repo de ton collègue).

L’app Flutter appelle le backend via **API_BASE_URL**, défini **au moment du build**. Il faut donc donner l’URL de ton backend Render (ex. `https://gameforge-backend-xxxx.onrender.com/api`) lors du build.

---

## Prérequis

- Backend déjà déployé sur Render (URL du type `https://gameforge-backend-xxxx.onrender.com`)
- Repo GitHub du front (ex. ton fork **GameForgeAi-fronts** ou **Game_front_deploy** selon comment tu as poussé)
- Compte Render connecté à GitHub

---

## Méthode 1 : Web Service avec Docker (recommandé)

Le dossier **GameForgeAi-fronts** contient un **Dockerfile** qui :

1. Build l’app Flutter en web avec `API_BASE_URL` passé au build
2. Sert les fichiers avec nginx en écoutant sur la variable **PORT** (exigée par Render)

### 1. Pousser le code sur GitHub

Si ce n’est pas déjà fait, pousse **GameForgeAi-fronts** (avec le Dockerfile, `nginx.conf.template`, `docker-entrypoint.sh`, `.dockerignore`) sur ton repo GitHub (ex. `fatimatouahmed4960-bit/GameForgeAi-fronts` ou le repo qui contient ce front).

### 2. Créer un Web Service sur Render

1. **Dashboard Render** → **New +** → **Web Service**.
2. Connecte le dépôt qui contient **GameForgeAi-fronts** (ou choisis le bon repo si le front est à la racine).
3. Renseigne :
   - **Name** : `gameforge-front` (ou autre).
   - **Region** : celle de ton choix.
   - **Branch** : `main` (ou la branche à déployer).
   - **Root Directory** : si le repo est uniquement le front Flutter, laisse **vide**. Si le repo contient un dossier (ex. `GameForgeAi-fronts`), mets **GameForgeAi-fronts**.
   - **Runtime** : **Docker** (obligatoire pour ce Dockerfile).
   - **Instance Type** : Free ou payant.

### 3. Variables d’environnement (Build)

Dans **Environment** → **Build**, ajoute une variable **Build-time** :

| Key             | Value (exemple)                                      |
|-----------------|------------------------------------------------------|
| `API_BASE_URL`  | `https://gameforge-backend-xxxx.onrender.com/api`   |

Remplace par l’URL réelle de ton backend Render (sans slash final, avec `/api`).

Optionnel (build) :

- `STRIPE_PUBLISHABLE_KEY` : si tu veux la fixer au build.
- `GOOGLE_SERVER_CLIENT_ID` : si tu utilises Google Sign-In et une autre valeur que celle par défaut.

### 4. Déploiement

- Clique **Create Web Service**.
- Render va construire l’image Docker (build Flutter + nginx). Le premier build peut prendre **plusieurs minutes** (téléchargement Flutter, `flutter build web`).
- Une fois le déploiement réussi, l’URL du front sera du type :  
  **https://gameforge-front-xxxx.onrender.com**

### 5. CORS backend

Sur le **backend** Render, configure **FRONTEND_URL** avec l’URL du front Render, par exemple :

`FRONTEND_URL=https://gameforge-front-xxxx.onrender.com`

---

## Méthode 2 : Build local puis Static Site (sans Docker)

Si tu préfères ne pas utiliser Docker sur Render :

1. **Build en local** (avec l’URL du backend) :
   ```bash
   cd Game_front_deploy/GameForgeAi-fronts
   flutter pub get
   flutter build web --release --dart-define=API_BASE_URL=https://gameforge-backend-xxxx.onrender.com/api
   ```
2. Le résultat est dans **build/web/**.
3. Sur Render : **New +** → **Static Site** → connecte un repo.
4. Pour que Render “voie” les fichiers buildés, deux options :
   - Soit tu commites **build/web/** dans une branche dédiée (ex. `deploy-static`) et tu pointes Render vers cette branche, avec **Publish Directory** = `build/web` (ou le chemin vers ce dossier selon la structure du repo).
   - Soit tu utilises un **script de build** sur Render : Render n’a pas Flutter en natif, donc cette option implique en pratique d’utiliser un **service de build externe** (ex. GitHub Actions) qui pousse les fichiers vers un repo/branche, puis Render Static Site publie ce dossier. Plus complexe.

En pratique, la **Méthode 1 (Docker)** est plus simple et ne nécessite pas de commiter les fichiers buildés.

---

## Récapitulatif Méthode 1 (Docker)

| Étape | Action |
|-------|--------|
| 1 | Repo GitHub à jour avec Dockerfile, nginx.conf.template, docker-entrypoint.sh |
| 2 | Render → New → Web Service → repo + **Runtime: Docker** |
| 3 | **Root Directory** : vide si repo = front, sinon `GameForgeAi-fronts` |
| 4 | Variable **API_BASE_URL** (Build) = URL du backend + `/api` |
| 5 | Create Web Service → attendre le build → tester l’URL du front |
| 6 | Backend : **FRONTEND_URL** = URL du front Render |

---

## Dépannage

- **Build Docker échoue** : vérifier les **Build Logs** (réseau, Flutter, `flutter build web`). Si le repo est dans un sous-dossier, **Root Directory** doit être renseigné.
- **Page blanche** : vérifier que **API_BASE_URL** a bien été passé au build (même valeur que le backend, avec `/api`).
- **Erreurs CORS** : mettre **FRONTEND_URL** sur le backend à l’URL exacte du front (sans slash final).
