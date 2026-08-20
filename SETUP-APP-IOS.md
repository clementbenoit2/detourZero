# Carnet de Route — app web pour iPhone

`index.html` est une app complète qui tourne entièrement dans Safari
(aucun serveur, aucun Node.js requis sur le téléphone) : elle appelle
directement l'API Google Maps JavaScript depuis le navigateur. Ajoutée à
l'écran d'accueil, elle s'ouvre en plein écran, sans barre d'adresse,
comme une app native.

## 1. Différence importante avec le script Node

Le script `find-stations-on-route.js` appelait l'API Google Maps
**depuis un serveur**, avec une clé tenue secrète. Cette app web appelle
l'API **depuis le navigateur** : la clé est donc visible dans le code
source de la page. Deux conséquences :

- Il faut activer une API supplémentaire : **Maps JavaScript API** (en
  plus de Directions API et Places API déjà activées).
- Il faut restreindre la clé **par référent HTTP (site web)** et non
  plus par IP, dans Google Cloud Console (Identifiants > votre clé >
  Restrictions relatives à l'application > Sites web). Ajoutez le
  domaine où vous hébergerez l'app (voir étape 3). Sans cette
  restriction, n'importe qui pourrait réutiliser votre clé.

Vous pouvez soit créer une deuxième clé dédiée à cette app (recommandé,
pour garder le script Node isolé), soit réutiliser la même en ajoutant
les deux types de restriction n'est pas possible sur une seule clé côté
Google : une clé est restreinte soit par IP, soit par référent HTTP, pas
les deux. D'où l'intérêt d'une clé séparée pour l'app web.

## 2. Configurer la clé dans le fichier

Ouvrez `index.html` dans un éditeur de texte, cherchez tout en bas :

```html
<script async src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY_ICI&libraries=places,geometry&callback=initApp"></script>
```

Remplacez `YOUR_API_KEY_ICI` par votre clé.

## 3. Héberger le fichier (nécessaire pour la géolocalisation)

Safari exige un contexte sécurisé (https) pour autoriser la
géolocalisation — ouvrir le fichier directement (`file://`) ne
fonctionnera pas pour la détection automatique de position (le champ de
saisie manuelle reste utilisable, mais autant avoir la détection auto).
Deux options gratuites et rapides :

- **GitHub Pages** : créez un dépôt GitHub, déposez `index.html` à la
  racine, activez Pages dans les paramètres du dépôt (branche
  `main`, dossier `/`). L'app sera accessible à une adresse du type
  `https://votre-nom.github.io/votre-depot/`.
- **Netlify Drop** (netlify.com/drop) : glissez-déposez simplement le
  fichier `index.html` sur la page, sans compte requis pour un test
  rapide. Une URL https est générée immédiatement.

Utilisez ensuite cette URL comme référent autorisé pour la clé API
(étape 1).

## 4. Ajouter l'app à l'écran d'accueil (iPhone/iPad)

1. Ouvrez l'URL https de votre app dans **Safari** (obligatoire — Chrome
   ou autres navigateurs sur iOS ne permettent pas l'ajout en mode
   plein écran).
2. Appuyez sur l'icône de partage (carré avec une flèche vers le haut).
3. Faites défiler et choisissez **"Sur l'écran d'accueil"**.
4. Confirmez le nom ("Carnet de Route") et validez.

Une icône apparaît sur votre écran d'accueil ; elle ouvre l'app en
plein écran, sans interface Safari.

## 5. Utilisation

- Au premier lancement, Safari demande l'autorisation de géolocalisation
  — acceptez pour que la position actuelle se remplisse automatiquement.
  Vous pouvez aussi la modifier ou la saisir manuellement
  (`latitude,longitude`).
- Renseignez la destination et la marque, choisissez le rayon de détour
  et éventuellement "Ouvertes uniquement à l'arrivée", puis appuyez sur
  **Rechercher**.
- Les résultats et les repères sur la carte se mettent à jour
  instantanément quand vous changez le rayon ou la case "ouvertes
  uniquement" ensuite : ces deux filtres ne relancent pas d'appel
  réseau, seule une nouvelle recherche (bouton Rechercher) le fait.

## 6. Coûts et limites

- Chaque recherche déclenche : 1 appel Directions (trajet), plusieurs
  appels Places Nearby Search (un tous les 2 km le long du trajet),
  1 appel Place Details et, pour les stations hors trajet, 1 appel
  Directions supplémentaire par station — jusqu'à 5 km de rayon
  maximum. Comptez large si vous cherchez une marque très présente sur
  un long trajet.
- Comme pour le script Node, l'algorithme de projection sur le trajet
  utilise le point le plus proche parmi les points de la polyline
  plutôt qu'une vraie projection point-segment : une approximation
  suffisante en pratique.
- L'app utilise l'API Places JavaScript "classique" (`PlacesService`),
  toujours fonctionnelle mais que Google fait évoluer vers une nouvelle
  version : à surveiller dans la documentation officielle si Google
  annonce une dépréciation.
