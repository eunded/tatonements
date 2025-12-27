# Message Banner - Composant Vue3 pour PWA

Un composant Vue3 autonome et hautement personnalisable pour afficher des messages d'information dans une Progressive Web App (PWA). Optimisé pour mobile et desktop.

## Fonctionnalités

- ✅ **Deux modes d'affichage** - Bannière fixe ou inline dans la page
- 🔄 **Système de fallback** - Basculement automatique entre URL primaire et secondaire
- 💾 **Persistance** - Mémorisation du dernier message lu via localStorage
- 📋 **Historique** - Consultation des messages précédents dans une modale scrollable
- ⏰ **Gestion d'expiration** - Affichage automatique selon les dates de validité
- 🎨 **Support Markdown** - Formatage simple (liens, gras, italique) ou complet
- 🎭 **Personnalisation CSS** - Classes personnalisables depuis le parent
- 📱 **Responsive** - Adaptation automatique mobile/desktop
- 🔌 **Slots** - Personnalisation du contenu du bouton
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

```bash
# Copier le composant
cp src/components/MessageBanner.vue /votre-projet/src/components/

# Installer les dépendances requises
npm install vue@^3.4.0 marked@^11.1.0
npm install -D sass@^1.69.0
```

## Utilisation

### Mode Bannière (position fixe)

```vue
<template>
  <MessageBanner
    primary-url="/messages-primary.json"
    secondary-url="/messages-secondary.json"
    :enable-markdown="true"
    display-mode="banner"
    position="top"
  />
</template>

<script>
import MessageBanner from '@/components/MessageBanner.vue'

export default {
  components: { MessageBanner }
}
</script>
```

### Mode Inline (dans la page)

```vue
<template>
  <div class="ma-section">
    <MessageBanner
      primary-url="/messages-primary.json"
      secondary-url="/messages-secondary.json"
      display-mode="inline"
      inline-class="mon-message-custom"
      :auto-open="false"
      :auto-open-when-empty="true"
    />
  </div>
</template>
```

**Note importante** : Pour passer des booléens, utilisez `:auto-open="false"` (avec `:`), pas `auto-open="false"` qui passerait la string `"false"`.

**Comportement autoOpen** :
- `autoOpen=false` : La bannière reste fermée au montage, seul le bouton est visible
- `autoOpen=true` + `autoOpenWhenEmpty=false` : Ouvre uniquement s'il y a des messages
- `autoOpen=true` + `autoOpenWhenEmpty=true` : Ouvre toujours, avec placeholder si vide
- Au clic sur le bouton de réouverture : **toujours** affiche la bannière (ignore les props)

### Personnalisation CSS

```vue
<template>
  <MessageBanner
    display-mode="banner"
    banner-class="ma-banniere"
    button-class="mes-boutons"
    reopen-button-class="mon-bouton-reouverture"
  >
    <template #reopen-button-content>
      📬 Nouveaux messages
    </template>
  </MessageBanner>
</template>

<style>
/* Personnaliser la bannière */
.ma-banniere {
  background: linear-gradient(to right, #ff6b6b, #ee5a6f) !important;
  border-bottom: 3px solid #c54752;
}

/* Personnaliser les boutons */
.mes-boutons {
  border-radius: 20px !important;
  text-transform: uppercase;
}

/* Positionner le bouton de réouverture */
.mon-bouton-reouverture {
  position: fixed !important;
  bottom: 20px !important;
  right: 20px !important;
  box-shadow: 0 4px 20px rgba(0,0,0,0.3) !important;
}
</style>
```

### Internationalisation (i18n)

Tous les textes sont personnalisables via props pour supporter plusieurs langues :

```vue
<template>
  <MessageBanner
    primary-url="/messages-primary.json"
    secondary-url="/messages-secondary.json"

    text-btn-read="✓ Read"
    text-btn-close="✕"
    text-btn-history="📋"
    text-tooltip-history="View history"
    text-tooltip-reopen="Show last message"
    text-history-title="Message History"
    text-history-read-badge="Read"
    text-history-empty="No messages in history"
    text-no-message-available="No messages available at the moment."
    error-message="Unable to load messages. Please try again later."
  >
    <template #reopen-button-content>
      💬 Messages
    </template>
  </MessageBanner>
</template>
```

**Note** : Tous les textes ont des valeurs par défaut en français. Les textes par défaut sont regroupés dans la constante `DEFAULT_TEXTS` en tête du fichier composant pour faciliter la maintenance.

## Props disponibles

| Prop | Type | Défaut | Description |
|------|------|--------|-------------|
| `primaryUrl` | String | *requis* | URL principale pour récupérer les messages |
| `secondaryUrl` | String | *requis* | URL de secours si la primaire échoue |
| **Mode d'affichage** | | | |
| `displayMode` | String | `'banner'` | Mode : `'banner'` (fixe) ou `'inline'` (dans la page) |
| `position` | String | `'top'` | Position si mode=banner : `'top'` ou `'bottom'` |
| **Contenu** | | | |
| `enableMarkdown` | Boolean | `false` | Activer le support Markdown |
| `fullMarkdown` | Boolean | `false` | Markdown complet (sinon simple : liens, gras, italique) |
| `maxHistory` | Number | `10` | Nombre de messages affichés dans l'historique |
| `errorMessage` | String | `'Impossible de charger...'` | Message affiché si les deux URLs échouent |
| **Comportement** | | | |
| `autoOpen` | Boolean | `true` | Ouvrir automatiquement la bannière au montage du composant |
| `autoOpenWhenEmpty` | Boolean | `true` | Si `autoOpen=true` et aucun message, afficher quand même un placeholder |
| **Personnalisation CSS** | | | |
| `bannerClass` | String | `''` | Classe CSS personnalisée pour la bannière (mode banner) |
| `inlineClass` | String | `''` | Classe CSS personnalisée pour le message inline (mode inline) |
| `inlineContainerClass` | String | `''` | Classe CSS personnalisée pour le conteneur inline |
| `buttonClass` | String | `''` | Classe CSS personnalisée pour les boutons (Lu, Historique, Fermer) |
| `reopenButtonClass` | String | `''` | Classe CSS personnalisée pour le bouton de réouverture |
| **Internationalisation (i18n)** | | | |
| `textBtnRead` | String | `'✓ Lu'` | Texte du bouton "Lu" |
| `textBtnClose` | String | `'✕'` | Texte du bouton "Fermer" |
| `textBtnHistory` | String | `'📋'` | Texte/icône du bouton "Historique" |
| `textBtnReopenDefault` | String | `'💬 Messages'` | Contenu par défaut du bouton réouverture (si pas de slot) |
| `textTooltipHistory` | String | `'Voir l\'historique'` | Tooltip du bouton historique |
| `textTooltipReopen` | String | `'Afficher le dernier message'` | Tooltip du bouton réouverture |
| `textHistoryTitle` | String | `'Historique des messages'` | Titre de la modale historique |
| `textHistoryReadBadge` | String | `'Lu'` | Badge "Lu" dans l'historique |
| `textHistoryEmpty` | String | `'Aucun message dans l\'historique'` | Message si historique vide |
| `textNoMessageAvailable` | String | `'Aucun message disponible...'` | Message placeholder si aucun message |
| `textDefaultError` | String | `'Impossible de charger...'` | Message d'erreur par défaut (alias de `errorMessage`) |

## Slots disponibles

| Slot | Description | Contenu par défaut |
|------|-------------|-------------------|
| `reopen-button-content` | Contenu du bouton de réouverture | `💬 Messages` |

**Exemple** :
```vue
<MessageBanner ...>
  <template #reopen-button-content>
    📬 <span>Nouveaux messages</span>
  </template>
</MessageBanner>
```

## Format des messages JSON

```json
{
  "messages": [
    {
      "id": "msg-001",
      "datetime": "2025-12-20T10:00:00Z",
      "expiryDate": "2026-01-20T23:59:59Z",
      "content": "Message avec **formatage** et [liens](https://example.com) 🎉"
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

Le fichier JSON doit être **strictement valide**. Erreurs courantes à éviter :

- ❌ Virgule finale après le dernier élément
- ❌ Guillemets simples au lieu de doubles (`'texte'` → `"texte"`)
- ❌ Commentaires (non supportés en JSON)
- ❌ Propriétés sans guillemets

**Validation recommandée** : jsonlint.com

## Modes d'affichage

### Mode Banner (bannière fixe)

Affiche les messages dans une bannière fixée en haut ou en bas de l'écran.

**Avantages** :
- Toujours visible
- Attire l'attention
- Parfait pour messages importants/urgents

**Configuration** :
```vue
<MessageBanner
  display-mode="banner"
  position="top"
  banner-class="ma-classe-custom"
/>
```

### Mode Inline (dans la page)

Affiche les messages directement à l'emplacement du composant dans le DOM.

**Avantages** :
- S'intègre naturellement dans le layout
- Moins intrusif
- Parfait pour messages contextuels

**Configuration** :
```vue
<MessageBanner
  display-mode="inline"
  inline-class="ma-classe-custom"
/>
```

## Personnalisation avancée

### Exemple complet avec toutes les options

```vue
<template>
  <div class="mon-app">
    <MessageBanner
      primary-url="/api/messages"
      secondary-url="/fallback/messages"
      
      display-mode="banner"
      position="bottom"
      
      :enable-markdown="true"
      :full-markdown="false"
      :max-history="20"

      error-message="⚠️ Serveur indisponible. Réessayez plus tard."
      :auto-open="true"
      :auto-open-when-empty="false"

      banner-class="custom-banner"
      button-class="custom-btn"
      reopen-button-class="custom-reopen"
    >
      <template #reopen-button-content>
        <svg><!-- icône custom --></svg>
        <span>Messages ({{ messageCount }})</span>
      </template>
    </MessageBanner>
  </div>
</template>

<style>
.custom-banner {
  background: var(--primary-color) !important;
  box-shadow: 0 -2px 10px rgba(0,0,0,0.2) !important;
}

.custom-btn {
  font-family: 'Montserrat', sans-serif !important;
  letter-spacing: 0.5px !important;
}

.custom-reopen {
  position: fixed !important;
  bottom: 80px !important;
  right: 20px !important;
  background: var(--accent-color) !important;
  animation: pulse 2s infinite !important;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}
</style>
```

## Gestion des erreurs

**Fichiers vides ou absents (404)** :
- Pas d'erreur levée (comportement normal)
- Message d'info affiché si `autoOpen=true` et `autoOpenWhenEmpty=true`
- Bannière fermée si `autoOpen=false` ou `autoOpenWhenEmpty=false`

**JSON invalide** :
- Tentative sur URL de fallback
- Message d'erreur si les deux échouent
- Message affiché selon `autoOpen` et `autoOpenWhenEmpty`
- Logs détaillés dans la console

**Erreurs réseau** :
- Basculement automatique URL primaire → secondaire
- Message d'erreur personnalisable via prop `errorMessage`
- Comportement d'affichage contrôlé par `autoOpen` et `autoOpenWhenEmpty`

**Au clic sur le bouton de réouverture** :
- La bannière s'ouvre **toujours** (ignore les props `autoOpen` et `autoOpenWhenEmpty`)
- Affiche un message s'il existe, sinon "Aucun message disponible pour le moment."

## Technologies utilisées

- **Vue 3** - Framework JavaScript (Composition API)
- **Vite** - Build tool
- **SCSS** - Préprocesseur CSS
- **Marked** - Parser Markdown

### Dépendances

**Production** :
- `vue@^3.4.0`
- `marked@^11.1.0`

**Développement** :
- `@vitejs/plugin-vue@^5.0.0`
- `sass@^1.69.0`
- `vite@^5.0.0`

## Migration depuis l'ancienne version

Si vous utilisiez la prop `floatingButton` :

**Avant** :
```vue
<MessageBanner :floating-button="true" />
```

**Maintenant** (deux options) :

1. **Bouton simple (recommandé)** :
```vue
<MessageBanner reopen-button-class="mon-style-bouton" />
```

2. **Bouton flottant avec CSS** :
```vue
<MessageBanner reopen-button-class="floating-btn" />

<style>
.floating-btn {
  position: fixed !important;
  bottom: 20px !important;
  right: 20px !important;
  border-radius: 50px !important;
}
</style>
```

## Compatibilité

- Vue 3.4+
- Navigateurs modernes (Chrome, Firefox, Safari, Edge)
- Support mobile iOS et Android
- PWA compatible

## Licence

MIT

---

**Note** : Ce composant utilise exclusivement la **Composition API** de Vue 3.
