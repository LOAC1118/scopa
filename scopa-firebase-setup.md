# 🔥 Setup Firebase pour Scopa Scoreboard

## Étape 1 — Créer le projet Firebase

1. Va sur [console.firebase.google.com](https://console.firebase.google.com)
2. Clique **"Ajouter un projet"** (ou "Create a project")
3. Nomme-le par exemple `scopa-scoreboard`
4. **Désactive Google Analytics** (pas nécessaire pour ce projet) → Continuer
5. Attends la création (~30 sec)

## Étape 2 — Activer Firestore Database

1. Dans le menu latéral → **Build → Firestore Database**
2. Clique **"Créer une base de données"**
3. Choisis **"Démarrer en mode production"** (on va configurer les règles juste après)
4. Choisis la région la plus proche : **eur3 (europe-west)** pour un usage en France
5. Clique **"Activer"**

## Étape 3 — Configurer les règles de sécurité

Dans Firestore → onglet **"Règles"** (Rules), remplace le contenu par :

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Collection scopa_games : lecture, création et suppression publiques
    match /scopa_games/{gameId} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['game_uuid', 'players', 'scores', 'winner_name'])
                    && request.resource.data.num_players is int
                    && request.resource.data.num_players >= 2
                    && request.resource.data.num_players <= 4;
      allow delete: if true;
      allow update: if false; // Pas de modification après création
    }
    
    // Collection scopa_tournaments : tournois
    match /scopa_tournaments/{tournamentId} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['name', 'format', 'participants', 'matches'])
                    && request.resource.data.participants.size() >= 2
                    && request.resource.data.participants.size() <= 16;
      allow delete: if true;
      allow update: if false;
    }
  }
}
```

**Clique "Publier"** pour appliquer les règles.

> ⚠️ Ces règles sont publiques (accès sans auth), comme demandé. Pour un usage famille, c'est parfait. Les règles ajoutent juste des validations basiques pour éviter le spam.

## Étape 4 — Enregistrer l'application Web

1. Sur la page d'accueil du projet, clique sur l'icône **Web** (`</>`) pour ajouter une app
2. Nom de l'app : `scopa-scoreboard`
3. **NE PAS** cocher "Firebase Hosting" (tu utilises GitHub Pages)
4. Clique **"Enregistrer l'app"**
5. Firebase te montre un objet `firebaseConfig` comme ceci :

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "scopa-scoreboard.firebaseapp.com",
  projectId: "scopa-scoreboard",
  storageBucket: "scopa-scoreboard.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123..."
};
```

**Copie ces valeurs**, tu vas les coller dans le HTML à l'étape suivante.

## Étape 5 — Coller la config dans le HTML

Ouvre `scopa-scoreboard-firebase.html` et cherche la section `CONFIGURATION FIREBASE` (autour de la ligne 560). Remplace les valeurs par les tiennes.

## Étape 6 — C'est prêt !

Ouvre le fichier dans ton navigateur ou pousse sur GitHub Pages. Les parties terminées seront automatiquement stockées dans Firestore et accessibles depuis n'importe quel appareil.

---

## 📊 Suivre tes données

Dans la console Firebase → **Firestore Database** → onglet **Data**, tu peux voir toutes les parties stockées dans la collection `scopa_games`, les supprimer manuellement si besoin, etc.

## 💰 Quotas gratuits (plan Spark)

Firestore gratuit couvre largement un usage familial :
- 50 000 lectures/jour
- 20 000 écritures/jour
- 1 GiB de stockage

Même avec des parties quotidiennes, tu n'approcheras jamais ces limites.
