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

```bash
# Installer les dépendances
npm install

# Lancer le serveur de développement
npm run dev

# Build pour la production
npm run build
```

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

## Fonctionnement

### 1. Récupération des messages

Le composant tente de récupérer les messages depuis l'URL primaire. En cas d'échec, il bascule automatiquement sur l'URL secondaire.

### 2. Filtrage par date

Seuls les messages dont la date d'expiration (`expiryDate`) est dans le futur sont affichés.

### 3. Messages non lus

Le composant affiche automatiquement le dernier message non lu. Une fois marqué comme lu, le message suivant non lu s'affiche (s'il existe).

### 4. Persistance

Les IDs des messages lus sont stockés dans le `localStorage` sous la clé `message-banner-read`. Ils restent mémorisés même après fermeture de l'application.

### 5. Réouverture

Un bouton flottant permet de rouvrir le dernier message à tout moment.

### 6. Historique

Un bouton dans la bannière ouvre une modale scrollable listant tous les messages (lus et non lus) jusqu'à la limite définie par `maxHistory`.

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
- **Composition API** - API moderne de Vue 3

## Structure du projet

```
tatonements/
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
