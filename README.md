# / Kwand

Kwand est l'outil interne Konversational pour trouver la date qui met tout le monde d'accord : soirée montage de la salle de sport, afterwork, atelier, n'importe quoi. On crée un événement, on partage le lien, chacun coche ses jours sur un calendrier, et la date qui réunit le plus de monde ressort d'elle-même.

Hébergé sur GitHub Pages, sans backend ni service externe : les données vivent dans `availability.json` directement dans ce repo (chaque action = un commit automatique via l'API GitHub).

## Contenu

- `index.html` : l'outil complet (HTML + CSS + JS, aucun build nécessaire)
- `availability.json` : les événements et disponibilités, mis à jour par le site
- `README.md` : ce guide

## Fonctionnalités

- Plusieurs événements en parallèle, chacun avec son lien direct partageable (`…/#/e/nom-de-l-evenement`)
- Création d'événement depuis l'interface : titre, description, période (jusqu'à 6 mois)
- Calendrier heatmap : plus une case est saumon, plus il y a de monde ce jour-là
- Top des dates, stats, pastilles à initiales, sélection perso en violet
- Prénom mémorisé sur l'appareil, dispos rechargées quand on revient
- Enregistrements simultanés fusionnés (personne n'écrase les dispos des autres)
- Responsive mobile

## Déploiement (10 minutes)

### 1. Créer le repo et publier les fichiers

1. Sur GitHub, crée un repo **public** (ex. `kwand`). Public est obligatoire pour GitHub Pages gratuit.
2. Uploade les 3 fichiers sur la branche `main` (bouton *Add file → Upload files*, pas besoin de git en local).

### 2. Activer GitHub Pages

1. Dans le repo : *Settings → Pages*.
2. Source : *Deploy from a branch*, branche `main`, dossier `/ (root)`, puis *Save*.
3. Après une ou deux minutes, le site est disponible sur `https://TON_COMPTE.github.io/kwand/`.

### 3. Créer le token (permet au site d'écrire dans le JSON)

1. GitHub → photo de profil → *Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token*.
2. Renseigne :
   - **Token name** : `kwand`
   - **Expiration** : 90 jours, à renouveler
   - **Resource owner** : ton compte (ou l'organisation si le repo y est)
   - **Repository access** : *Only select repositories* → sélectionne uniquement `kwand`
   - **Permissions → Repository permissions → Contents** : *Read and write*. Rien d'autre.
3. Génère et copie le token (`github_pat_...`).

### 4. Configurer le token dans index.html

Le token ne doit pas apparaître en clair dans le code, sinon le secret scanning de GitHub le révoque automatiquement. On l'encode en base64 et on le coupe en deux :

1. Ouvre la console de ton navigateur (F12 → Console) et tape :
   ```js
   btoa("github_pat_COLLE_TON_TOKEN_ICI")
   ```
2. Copie le résultat (une longue chaîne) et coupe-le en deux morceaux à peu près égaux, à n'importe quel endroit.
3. Dans `index.html`, bloc `CONFIG` en haut du `<script>`, renseigne :
   ```js
   owner: 'ton-compte-github',
   repo: 'kwand',
   branch: 'main',
   tokenParts: ['premiere_moitie', 'seconde_moitie']
   ```
4. Commit. Le site passe automatiquement en mode lecture + écriture (le bandeau "lecture seule" disparaît).

Sans token, le site fonctionne en lecture seule (consultation uniquement, ni création ni enregistrement).

### 5. Partager

Envoie l'URL à l'équipe, ou le lien direct d'un événement (`…/#/e/soiree-montage-salle-de-sport`). Aucun compte à créer pour les participants.

## Gestion des événements

- **Créer** : bouton "+ Créer un événement" sur la page d'accueil.
- **Supprimer ou modifier** : volontairement absent de l'interface (pour éviter qu'un clic malheureux efface les dispos de tout le monde). Édite `availability.json` directement sur GitHub : supprime le bloc de l'événement dans `events`, ou modifie son titre / sa description / ses mois. L'historique git garde toutes les versions.

Structure du JSON :

```json
{
  "tool": "Kwand",
  "updated": "2026-07-31T10:00:00Z",
  "events": {
    "soiree-montage-salle-de-sport": {
      "title": "Soirée montage salle de sport",
      "description": "Optionnelle",
      "months": ["2026-08", "2026-09"],
      "created": "2026-07-31T00:00:00Z",
      "people": {
        "Enzo": ["2026-08-03", "2026-09-07"]
      }
    }
  }
}
```

## Sécurité : à savoir avant de partager l'URL

Ce montage est un compromis assumé pour un petit outil interne, gratuit et sans service externe :

- **Le token est reconstituable par quiconque lit le code source.** Il est limité à ce seul repo avec la permission Contents uniquement. Pire cas : quelqu'un modifie le contenu de ce repo, et tout se restaure via l'historique git. Aucun accès aux autres repos.
- **Le repo est public** : titres d'événements, prénoms et disponibilités sont visibles par n'importe qui. Prénoms uniquement, pas de noms complets, et pas d'événement confidentiel.
- **Mets une date d'expiration au token** et renouvelle-le (étape 4, deux minutes).
- Ne partage l'URL qu'en interne.

Si un jour le besoin dépasse ce cadre (données sensibles, audience large), bascule vers un vrai backend (Firebase, Supabase) plutôt que d'étendre ce montage.

## Fonctionnement technique

- Lecture : `GET /repos/{owner}/{repo}/contents/availability.json` (API GitHub, CORS ouvert).
- Écriture : `PUT` sur le même endpoint avec le `sha` courant. En cas de conflit (deux personnes enregistrent en même temps), le site relit le fichier et refusionne, jusqu'à 3 tentatives. Chaque action ne touche que sa propre tranche du JSON : les dispos d'une personne dans un événement, ou l'ajout d'un événement.
- Rafraîchissement automatique toutes les 90 secondes quand le token est configuré (quota API : 5 000 requêtes/heure avec token).

---
*© Konversational*
