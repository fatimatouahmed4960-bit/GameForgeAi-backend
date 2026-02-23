# Déployer le front GameForge (Flutter Web) sur Vercel

Projet : **Game_front_deploy/GameForgeAi-fronts** (Flutter Web).

L’app Flutter appelle le backend via **API_BASE_URL**, défini **au moment du build**. Tu dois configurer l’URL de ton backend (ex. Render ou autre) dans les variables d’environnement Vercel.

---

## Prérequis

- Backend déjà déployé (URL du type `https://gameforge-backend-xxxx.onrender.com/api` ou autre)
- Repo GitHub du front (ex. **GameForgeAi-fronts**)
- Compte [Vercel](https://vercel.com) (connexion GitHub)

---

## 1. Pousser le code sur GitHub

Assure-toi que le repo contient :

- Le projet Flutter (**GameForgeAi-fronts** si le repo est au niveau parent, ou à la racine si le repo = front uniquement)
- Le fichier **vercel.json** à la racine du projet buildé (donc à la racine du repo si le repo = front, sinon dans **GameForgeAi-fronts/**)

---

## 2. Créer le projet sur Vercel

1. Va sur **[vercel.com](https://vercel.com)** → **Add New** → **Project**.
2. **Import** le dépôt GitHub qui contient le front (ex. ton fork **GameForgeAi-fronts**).
3. Si le repo contient un sous-dossier (ex. **Game_front_deploy** avec **GameForgeAi-fronts** dedans) :
   - **Root Directory** : choisis le dossier du front, ex. **GameForgeAi-fronts** (ou **Game_front_deploy/GameForgeAi-fronts** selon la structure).
4. **Framework Preset** : laisse **Other** (pas de preset Flutter).
5. Vercel peut détecter **vercel.json** : les champs **Build Command**, **Install Command**, **Output Directory** seront alors pris depuis ce fichier. Sinon, renseigne-les à la main (voir ci‑dessous).

---

## 3. Variables d’environnement (Build)

Dans **Settings** du projet → **Environment Variables** :

| Name            | Value (exemple)                                      | Environnement |
|-----------------|------------------------------------------------------|----------------|
| `API_BASE_URL`  | `https://gameforge-backend-xxxx.onrender.com/api`    | Production (et Preview si tu veux) |

Remplace par l’URL réelle de ton backend, **avec** le préfixe `/api` à la fin, **sans** slash final.

Optionnel :

- `STRIPE_PUBLISHABLE_KEY` : clé publique Stripe.
- `GOOGLE_SERVER_CLIENT_ID` : si tu changes le client Google OAuth.

---

## 4. Build Command, Install Command, Output Directory

Si tu ne mets **pas** de **vercel.json** (ou si tu préfères tout configurer dans l’interface) :

- **Install Command** :
  ```bash
  if [ -d flutter ]; then (cd flutter && git pull); else git clone https://github.com/flutter/flutter.git --depth 1; fi && flutter/bin/flutter doctor && flutter/bin/flutter config --enable-web && flutter/bin/flutter pub get
  ```
- **Build Command** :
  ```bash
  flutter/bin/flutter build web --release --dart-define=API_BASE_URL=$API_BASE_URL
  ```
- **Output Directory** : `build/web`

Avec **vercel.json** présent à la racine du projet importé, ces valeurs sont déjà définies dans le fichier.

---

## 5. Déployer

- Clique **Deploy**.
- Le premier build peut prendre **plusieurs minutes** (clone de Flutter, `flutter doctor`, `flutter build web`).
- À la fin, tu obtiens une URL du type : **https://gameforge-front-xxxx.vercel.app** (ou le nom de projet que tu as choisi).

Chaque push sur la branche connectée déclenchera un nouveau déploiement (si l’auto-deploy est activé).

---

## 6. CORS côté backend

Sur le **backend** (Render ou autre), configure **FRONTEND_URL** avec l’URL du front Vercel, par exemple :

```env
FRONTEND_URL=https://gameforge-front-xxxx.vercel.app
```

(sans slash final). Cela permet au backend d’accepter les requêtes du front (CORS) et d’utiliser cette URL dans les liens (ex. reset password).

---

## Récapitulatif

| Étape | Action |
|-------|--------|
| 1 | Repo GitHub à jour avec **vercel.json** à la racine du projet Flutter |
| 2 | Vercel → Add New → Project → importer le repo |
| 3 | **Root Directory** : dossier du front (ex. `GameForgeAi-fronts`) si besoin |
| 4 | **Environment Variables** : `API_BASE_URL` = URL du backend + `/api` |
| 5 | Deploy → attendre le build → tester l’URL Vercel |
| 6 | Backend : **FRONTEND_URL** = URL du front Vercel |

---

## Dépannage

- **Build échoue (Flutter introuvable)** : vérifier que **Install Command** est bien exécuté (logs du build) et que **Root Directory** pointe vers le dossier qui contient `pubspec.yaml` et `vercel.json`.
- **Page blanche** : vérifier que **API_BASE_URL** est défini en variable d’environnement Vercel et qu’il correspond à l’URL réelle du backend (avec `/api`).
- **Erreurs CORS** : vérifier **FRONTEND_URL** sur le backend (exactement l’URL du front Vercel, sans slash final).
- **404 sur les routes** : Vercel sert du static ; pour le routing côté client (go_router), il faut rediriger toutes les routes vers `index.html`. Ajouter dans **vercel.json** une section **rewrites** (voir ci‑dessous).

---

## Routing (SPA) : rewrites

Pour que les routes Flutter (ex. `/dashboard`, `/profile`) fonctionnent au rechargement ou à l’accès direct, il faut que toutes les requêtes soient renvoyées vers `index.html`. Ajoute dans **vercel.json** :

```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

Si tu as déjà d’autres clés dans **vercel.json**, fusionne-les (une seule racine `{}`).
