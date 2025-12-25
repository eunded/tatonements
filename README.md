# Message Banner - Composant Vue3 pour PWA

Un composant Vue3 autonome et configurable pour afficher des messages d'information dans une Progressive Web App (PWA). Optimisé pour mobile et desktop.

## Fonctionnalités

- ✅ **Affichage de bannières** - Messages courts et clairs
- 🔄 **Système de fallback** - Basculement automatique entre URL primaire et secondaire
- 💾 **Persistance** - Mémorisation des messages lus via localStorage
- 📋 **Historique** - Consultation des messages précédents dans une modale scrollable
- ⏰ **Gestion d'expiration** - Affichage automatique selon les dates de validité
- 🎨 **Support Markdown** - Formatage simple (liens, gras, italique) ou complet
- 📱 **Responsive** - Adaptation automatique mobile/desktop
- 🎯 **Position paramétrable** - Bannière en haut ou en bas
- 🔗 **URLs cliquables** - Détection et activation automatique des liens

## Installation

### Installation complète du projet

```bash
# Installer les dépendances
npm install

# Lancer le serveur de développement
npm run dev

# Build pour la production
npm run build
```

### Utilisation du composant dans un projet existant

Si vous souhaitez intégrer uniquement le composant `MessageBanner.vue` dans votre projet Vue3 existant :

1. **Copier le fichier du composant** :
   ```bash
   # Copier MessageBanner.vue dans votre projet
   cp src/components/MessageBanner.vue /votre-projet/src/components/
   ```

2. **Installer les dépendances requises** :
   ```bash
   # Vue 3 (si pas déjà installé)
   npm install vue@^3.4.0

   # Marked (pour le support Markdown)
   npm install marked@^11.1.0

   # SASS (pour les styles SCSS)
   npm install -D sass@^1.69.0
   ```

3. **Importer et utiliser le composant** :
   ```vue
   <script>
   import MessageBanner from '@/components/MessageBanner.vue'

   export default {
     components: {
       MessageBanner
     }
   }
   </script>
   ```

**Note** : Le composant nécessite **Marked** pour le support Markdown. Si vous n'utilisez pas le Markdown (`enableMarkdown: false`), l'installation de Marked reste nécessaire car le composant l'importe. Pour éviter cette dépendance, vous devriez modifier le composant pour importer Marked conditionnellement.

## Utilisation

### Intégration basique

```vue
<template>
  <MessageBanner
    primary-url="/messages-primary.json"
    secondary-url="/messages-secondary.json"
    :enable-markdown="true"
    position="top"
  />
</template>

<script>
import MessageBanner from './components/MessageBanner.vue'

export default {
  components: {
    MessageBanner
  }
}
</script>
```

### Props disponibles

| Prop | Type | Défaut | Description |
|------|------|--------|-------------|
| `primaryUrl` | String | *requis* | URL principale pour récupérer les messages |
| `secondaryUrl` | String | *requis* | URL de secours si la primaire échoue |
| `enableMarkdown` | Boolean | `false` | Activer le support Markdown |
| `fullMarkdown` | Boolean | `false` | Markdown complet (sinon simple : liens, gras, italique) |
| `maxHistory` | Number | `10` | Nombre de messages affichés dans l'historique |
| `position` | String | `'top'` | Position de la bannière : `'top'` ou `'bottom'` |
| `floatingButton` | Boolean | `false` | Bouton de réouverture flottant (sinon barre normale) |
| `errorMessage` | String | `'Impossible de charger...'` | Message affiché si les deux URLs échouent |
| `showNoMessageInfo` | Boolean | `true` | Afficher un message d'information si aucun message disponible |

## Format des messages JSON

Les messages sont récupérés depuis un fichier JSON avec la structure suivante :

```json
{
  "messages": [
    {
      "id": "msg-001",
      "datetime": "2025-12-20T10:00:00Z",
      "expiryDate": "2026-01-20T23:59:59Z",
      "content": "Votre message avec **formatage** et [liens](https://example.com) 🎉"
    }
  ]
}
```

### Champs obligatoires

- **id** : Identifiant unique du message (String)
- **datetime** : Date/heure d'émission au format ISO 8601 (String)
- **expiryDate** : Date/heure d'expiration au format ISO 8601 (String)
- **content** : Contenu du message (String, peut contenir Markdown)

### ⚠️ Validation du format JSON

Le fichier JSON doit être **strictement valide**. Les erreurs courantes à éviter :

- ❌ Virgule finale après le dernier élément d'un tableau ou objet
- ❌ Guillemets simples au lieu de doubles (`'texte'` → `"texte"`)
- ❌ Commentaires (non supportés en JSON)
- ❌ Propriétés sans guillemets (`id:` → `"id":`)

**Validation recommandée** : Utilisez un validateur JSON en ligne (jsonlint.com) ou votre éditeur de code avant de déployer.

Si le fichier JSON est invalide, le composant affichera le message d'erreur configuré via la prop `errorMessage` et tentera l'URL de fallback.

## Fonctionnement

### 1. Récupération des messages

Le composant tente de récupérer les messages depuis l'URL primaire. En cas d'échec, il bascule automatiquement sur l'URL secondaire.

### 2. Filtrage par date

Seuls les messages dont la date d'expiration (`expiryDate`) est dans le futur sont affichés.

### 3. Messages non lus

Le composant affiche automatiquement le dernier message non lu. Une fois marqué comme lu, le message suivant non lu s'affiche (s'il existe).

### 4. Persistance

Le dernier message lu est stocké dans le `localStorage` sous la clé `message-banner-last-read`. Seul l'ID du dernier message lu est conservé pour optimiser le stockage. Les messages plus anciens que le dernier lu sont automatiquement considérés comme lus.

### 5. Réouverture

Deux modes disponibles pour rouvrir le dernier message :
- **Mode normal** (par défaut) : Une barre discrète en haut ou en bas avec le texte "Message disponible"
- **Mode flottant** : Un bouton rond flottant sur le côté droit de l'écran (avec `floatingButton: true`)

### 6. Historique

Un bouton dans la bannière ouvre une modale scrollable listant tous les messages (lus et non lus) jusqu'à la limite définie par `maxHistory`.

### 7. Gestion des erreurs

Le composant gère plusieurs types d'erreurs :

**Erreurs réseau** :
- Si l'URL primaire échoue, basculement automatique sur l'URL secondaire
- Si les deux URLs échouent, affichage du message d'erreur personnalisable

**Erreurs de parsing JSON** :
- Si le JSON est invalide (erreur de syntaxe), traitement comme une erreur réseau
- Tentative sur l'URL de fallback
- Message d'erreur affiché si les deux fichiers sont invalides
- Logs détaillés dans la console pour faciliter le débogage

**Fichiers vides ou sans messages** :
- Si le fichier est accessible mais vide (littéralement vide), aucune erreur n'est levée
- Si le fichier contient un JSON valide mais aucun message, deux comportements possibles :
  - `showNoMessageInfo: true` (défaut) : Affiche "Aucun message disponible pour le moment."
  - `showNoMessageInfo: false` : Aucune bannière affichée
- Le bouton de réouverture reste accessible pour vérifier s'il y a de nouveaux messages

**Configuration** :
- Le message d'erreur est configurable via la prop `errorMessage`
- Le message d'information "aucun message" peut être désactivé avec `showNoMessageInfo: false`
- Les erreurs sont loguées dans la console avec des détails précis

## Support Markdown

### Mode simple (par défaut)

- **Gras** : `**texte**`
- *Italique* : `*texte*`
- [Liens](url) : `[texte](url)`

### Mode complet

Active toutes les fonctionnalités de Markdown via la bibliothèque `marked` :
- Titres, listes, citations
- Code inline et blocs
- Tableaux
- Et plus encore...

## Exemples de messages

### Message simple

```json
{
  "id": "welcome-001",
  "datetime": "2025-12-25T10:00:00Z",
  "expiryDate": "2026-01-31T23:59:59Z",
  "content": "Bienvenue sur notre application ! 👋"
}
```

### Message avec lien

```json
{
  "id": "update-001",
  "datetime": "2025-12-25T10:00:00Z",
  "expiryDate": "2026-01-31T23:59:59Z",
  "content": "Nouvelle version disponible ! Consultez les [notes de version](https://example.com/changelog)."
}
```

### Message avec formatage

```json
{
  "id": "important-001",
  "datetime": "2025-12-25T10:00:00Z",
  "expiryDate": "2026-01-31T23:59:59Z",
  "content": "**Important** : Maintenance programmée le 30 décembre. *L'application sera indisponible pendant 2 heures.*"
}
```

## Personnalisation

### Modifier les couleurs

Éditez le fichier `MessageBanner.vue` et ajustez les variables SCSS :

```scss
// Gradient de la bannière
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

// Couleur du bouton principal
.btn-primary {
  background: white;
  color: #667eea;
}
```

### Ajuster le style

Le composant utilise SCSS avec un style modulaire. Toutes les classes sont scopées pour éviter les conflits.

## Technologies utilisées

- **Vue 3** - Framework JavaScript réactif
- **Vite** - Build tool ultra-rapide
- **SCSS** - Préprocesseur CSS
- **Marked** - Parser Markdown
- **Composition API** - API moderne de Vue 3 (setup(), ref(), computed(), onMounted())

### Dépendances

Le composant requiert les dépendances suivantes :

**Production** :
- `vue@^3.4.0` - Framework Vue 3
- `marked@^11.1.0` - Parser Markdown (requis même si `enableMarkdown: false`)

**Développement** :
- `@vitejs/plugin-vue@^5.0.0` - Plugin Vite pour Vue
- `sass@^1.69.0` - Compilateur SCSS
- `vite@^5.0.0` - Build tool

Voir le fichier `package.json` pour les versions exactes.

### Note sur l'API Composition

Le composant utilise exclusivement la **Composition API** de Vue 3, avec :
- `setup(props)` pour la logique du composant
- `ref()` pour les états réactifs
- `computed()` pour les propriétés calculées
- `onMounted()` pour le cycle de vie
- Pas d'Options API (data, methods, etc.)

## Structure du projet

```
message-banner/
├── public/
│   ├── messages-primary.json    # Messages principaux
│   ├── messages-secondary.json  # Messages de secours
│   └── manifest.json            # Manifest PWA
├── src/
│   ├── components/
│   │   └── MessageBanner.vue    # Composant principal
│   ├── App.vue                  # Application de démo
│   ├── main.js                  # Point d'entrée
│   └── style.scss               # Styles globaux
├── index.html
├── package.json
├── vite.config.js
└── README.md
```

## Compatibilité

- Vue 3.4+
- Navigateurs modernes (Chrome, Firefox, Safari, Edge)
- Support mobile iOS et Android
- PWA compatible

## Licence

MIT

## Auteur

Créé avec Vue3 et Vite pour une utilisation en PWA mobile/desktop.
