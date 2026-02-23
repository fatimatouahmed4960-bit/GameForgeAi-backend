# Analyse du code – Game_front_deploy

## 1. Vue d’ensemble

**Game_front_deploy** contient **GameForgeAi-fronts** : une application **Flutter** (mobile + web) pour la plateforme GameForge AI (création de jeux, templates, IA, abonnements, etc.). C’est le front qui consomme l’API du backend GameForgeAi-backend.

- **Stack** : Flutter, Dart SDK ^3.10.7
- **Plateformes** : Android, iOS, Web, Windows, macOS, Linux
- **State** : Provider, Riverpod
- **Navigation** : go_router

---

## 2. Structure du projet

```
Game_front_deploy/
└── GameForgeAi-fronts/
    ├── lib/
    │   ├── main.dart                    # Point d'entrée, Stripe init, AuthProvider, router
    │   ├── core/
    │   │   ├── constants/               # Couleurs, typo, espacements, ombres
    │   │   ├── guards/                  # AuthGuard (protection des routes)
    │   │   ├── providers/               # AuthProvider, ThemeProvider, BuildMonitorProvider
    │   │   ├── router/                  # go_router (app_router.dart)
    │   │   ├── services/                # API, auth, billing, assets, templates, notifications, coach, etc.
    │   │   ├── themes/                  # AppTheme (light/dark)
    │   │   └── utils/
    │   └── presentation/
    │       ├── screens/                 # Écrans par domaine
    │       │   ├── onboarding/          # Splash, welcome, features, permissions
    │       │   ├── auth/                 # Login, signup
    │       │   ├── dashboard/            # Home dashboard
    │       │   ├── project/             # Projets, IA, config, coach, WebGL, export
    │       │   ├── build/               # Build config, progress, results
    │       │   ├── marketplace/         # Templates, upload, détails
    │       │   ├── profile/             # Profil, abo, paramètres, sécurité
    │       │   ├── notifications/       # Notifications
    │       │   ├── assets/              # Bibliothèque assets, export
    │       │   └── quiz/
    │       └── widgets/                 # Composants réutilisables + CoachGlobalOverlay
    ├── web/                             # Web (index.html, manifest.json)
    ├── android/, ios/, windows/, macos/, linux/
    ├── assets/                          # images/, animations/, icons/
    └── pubspec.yaml
```

---

## 3. Configuration et API backend

L’URL de l’API est définie **au moment du build** via une variable d’environnement Dart :

- **Fichier** : `lib/core/services/api_service.dart`
- **Variable** : `API_BASE_URL` (compilation)
  - Défaut : `http://127.0.0.1:3000/api`
  - Sur Android (émulateur) : si défaut, utilisation de `http://10.0.2.2:3000/api`
  - Pour la prod : passer à la compilation :  
    `--dart-define=API_BASE_URL=https://ton-backend.onrender.com/api`

Autres variables optionnelles (compilation) :

- **STRIPE_PUBLISHABLE_KEY** : clé publique Stripe (sinon récupérée depuis l’API backend au démarrage)
- **GOOGLE_SERVER_CLIENT_ID** : OAuth Google (backend web)
- **GOOGLE_MACOS_CLIENT_ID** : OAuth Google (macOS)

---

## 4. Fonctionnalités principales (écrans / services)

| Domaine | Contenu |
|--------|---------|
| **Onboarding** | Splash, welcome, features, permissions |
| **Auth** | Login, signup (email + rôle), Google Sign-In |
| **Dashboard** | Accueil après connexion |
| **Projets** | Liste, détail, création depuis template ou IA, config IA, coach, lecture WebGL, export |
| **Build** | Configuration build, progression, résultats |
| **Marketplace** | Liste templates, détail, achat, upload template |
| **Profil** | Profil, abonnement, paramètres, sécurité, CGU, confidentialité, à propos |
| **Notifications** | Liste, messages in-app |
| **Assets** | Bibliothèque d’assets, export |

Services côté app : **ApiService**, **AuthService**, **BillingService**, **TemplatesService**, **ProjectsService**, **NotificationsService**, **AssetsService**, **CoachOverlayController**, **LocalNotificationsService**, etc.

---

## 5. Dépendances notables (pubspec.yaml)

- **Navigation** : go_router
- **State** : provider, flutter_riverpod
- **HTTP** : http
- **Auth** : google_sign_in, flutter_secure_storage, local_auth
- **UI** : lottie, cached_network_image, shimmer, font_awesome_flutter, flutter_stripe
- **Média** : image_picker, image_cropper, video_player, webview_flutter
- **Autres** : socket_io_client, speech_to_text, flutter_tts, share_plus, qr_flutter, etc.

---

## 6. Déploiement web (résumé)

- **Build** : `flutter build web`
- **URL API en prod** : obligatoire de passer `--dart-define=API_BASE_URL=https://...` au build.
- **Sortie** : dossier `build/web/` (fichiers statiques). À héberger sur un serveur web ou un hébergeur statique (Render Static Site, Netlify, Firebase, Vercel, etc.).
- **Stripe** : optionnel en dart-define (`STRIPE_PUBLISHABLE_KEY`) ; sinon récupéré depuis le backend au lancement.

Le fichier **DEPLOY_VERCEL.md** (dans ce dossier) décrit la méthode pas à pas pour déployer ce front sur **Vercel**.
