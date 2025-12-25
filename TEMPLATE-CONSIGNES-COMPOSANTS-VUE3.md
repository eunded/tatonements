# Template de Consignes pour Composants Vue3

## 📋 Checklist Critique (à remplir avant de démarrer)

### 1. Architecture & API

- [ ] **API Style** : Composition API / Options API / les deux ?
- [ ] **Build tool** : Vite / Vue CLI / Webpack / autre ?
- [ ] **TypeScript** : Oui / Non ?
- [ ] **Styling** : SCSS / CSS / CSS Modules / Tailwind / autre ?

### 2. Modes d'Affichage

- [ ] **Un seul mode d'affichage** OU **plusieurs modes** ?
  - Si plusieurs : lister tous les modes
  - Exemple : `'banner'` (fixed), `'inline'` (dans le flow), `'modal'`, etc.

- [ ] **Positionnement** (si fixed) : top / bottom / left / right / custom ?

### 3. Personnalisation & Extensibilité

- [ ] **CSS doit être personnalisable depuis le parent** ?
  - ✅ Props de classes CSS : `bannerClass`, `buttonClass`, etc.
  - ❌ Styles uniquement dans le composant

- [ ] **Contenu personnalisable** ?
  - ✅ Slots Vue pour injection de contenu
  - Liste des slots nécessaires : _______________

- [ ] **Comportement personnalisable** ?
  - Props pour activer/désactiver features
  - Événements émis : _______________

### 4. Gestion des Données

#### Fetch / API

- [ ] **Source de données** : URL / Props / Vuex / Pinia / autre ?
- [ ] **Fallback** : URL secondaire / données par défaut / autre ?
- [ ] **Erreurs considérées "normales"** :
  - [ ] 404 (fichier absent)
  - [ ] Fichier vide
  - [ ] Timeout
  - [ ] Autre : _______________

- [ ] **Vraies erreurs** (à afficher à l'utilisateur) :
  - [ ] Parsing JSON
  - [ ] Erreurs réseau (pas de connexion)
  - [ ] HTTP 5xx (erreur serveur)
  - [ ] Autre : _______________

#### Persistance

- [ ] **localStorage / sessionStorage / cookies / autre** ?
- [ ] **Données à persister** :
  - [ ] Tout l'historique
  - [ ] Uniquement dernier état (optimisé)
  - [ ] Préférences utilisateur
  - [ ] Autre : _______________

- [ ] **Clé(s) localStorage** : _______________

### 5. Contenu & Format

- [ ] **Format des données** : JSON / XML / RSS / autre ?
- [ ] **Structure exacte** (fournir exemple) :
```json
{
  "items": [
    {
      "id": "...",
      "date": "...",
      // ...
    }
  ]
}
```

- [ ] **Support Markdown** : Non / Simple (liens, gras) / Complet / Custom ?
- [ ] **Support HTML** : Non / Sanitisé / Complet ?
- [ ] **Emojis** : Oui / Non ?

### 6. Interactions Utilisateur

- [ ] **Actions disponibles** :
  - [ ] Fermer/masquer
  - [ ] Marquer comme lu
  - [ ] Voir historique
  - [ ] Autre : _______________

- [ ] **Bouton de réouverture** :
  - Type : Inline / Fixed / Floating / Aucun
  - Personnalisable : Oui / Non
  - Via : Slot / Props / Classe CSS

### 7. Responsive & Compatibilité

- [ ] **Cibles** : Desktop / Mobile / Tablette / PWA / toutes ?
- [ ] **Breakpoints** : _______________
- [ ] **Compatibilité navigateurs** : Modernes / IE11+ / autre ?

### 8. Cas Particuliers

- [ ] **Comportement si aucune donnée** :
  - Afficher message info : Oui / Non
  - Message personnalisable : Oui / Non
  - Masquer complètement : Oui / Non

- [ ] **Comportement si erreur** :
  - Afficher message erreur : Oui / Non
  - Message personnalisable : Oui / Non
  - Retry automatique : Oui / Non

---

## 🎯 Template de Consignes (exemple rempli)

```markdown
# Composant [NOM] - Vue3

## Architecture
- Vue 3 Composition API (obligatoire, pas Options API)
- Build: Vite
- Styling: SCSS avec scoped styles
- PWA compatible (mobile + desktop)

## Modes d'affichage
Le composant supporte 2 modes via prop `displayMode`:
1. **'banner'** : Position fixed (top ou bottom via prop `position`)
2. **'inline'** : S'affiche dans le flow DOM à l'emplacement du composant

## Personnalisation
### CSS (obligatoire)
Props de classes CSS pour styling depuis le parent SANS ouvrir le composant:
- `bannerClass` : Customiser la bannière (mode banner)
- `inlineClass` : Customiser le message (mode inline)
- `inlineContainerClass` : Customiser le conteneur (mode inline)
- `buttonClass` : Tous les boutons d'action
- `reopenButtonClass` : Bouton de réouverture spécifiquement

### Contenu
Slots Vue pour personnalisation:
- `reopen-button-content` : Contenu du bouton de réouverture
  - Défaut: "💬 Messages"

## Données & Fetch
### Source
- Prop `primaryUrl` : URL principale (requise)
- Prop `secondaryUrl` : URL fallback (requise)
- Format: JSON avec structure:
```json
{
  "messages": [
    {
      "id": "string unique",
      "datetime": "ISO 8601",
      "expiryDate": "ISO 8601",
      "content": "string avec Markdown"
    }
  ]
}
```

### Gestion erreurs
**CAS NORMAUX (pas d'erreur affichée):**
- 404 sur les deux URLs
- Fichiers vides (texte vide ou `""`)
- Messages tous expirés

**VRAIES ERREURS (afficher message):**
- Parsing JSON invalide
- Erreurs réseau (fetch failed)
- HTTP 5xx

Prop `showNoMessageInfo` (Boolean, default: true):
- `true` : Affiche messages info ET erreurs
- `false` : Masque tout (sauf vrais messages valides)

## Persistance
localStorage avec clé `message-banner-last-read`:
- Stocker UNIQUEMENT l'ID du dernier message lu
- Logique: afficher messages avec ID > lastReadId
- PAS de Set/Array (optimisation stockage)

## Contenu
- Prop `enableMarkdown` (Boolean) : Activer Markdown
- Prop `fullMarkdown` (Boolean) :
  - `false` (défaut): Simple (liens, **gras**, *italique*)
  - `true`: Complet via library "marked"
- Emojis supportés nativement

## Interactions
- Bouton "Fermer" : Masque le message actuel
- Bouton "Lu" : Marque comme lu + passe au suivant
- Bouton "Historique" : Ouvre modale scrollable (max `maxHistory` items)
- Bouton réouverture : Simple bouton inline (PAS fixed bar), personnalisable via slot

## Responsive
- Mobile first
- Breakpoint : 768px (tablette/desktop)
- Touch-friendly (boutons min 44px)

## Props complètes

| Prop | Type | Défaut | Description |
|------|------|--------|-------------|
| `primaryUrl` | String | *requis* | URL principale |
| `secondaryUrl` | String | *requis* | URL secondaire |
| `displayMode` | String | `'banner'` | `'banner'` ou `'inline'` |
| `position` | String | `'top'` | `'top'` ou `'bottom'` (si mode=banner) |
| `enableMarkdown` | Boolean | `false` | Support Markdown |
| `fullMarkdown` | Boolean | `false` | Markdown complet vs simple |
| `maxHistory` | Number | `10` | Nombre max dans historique |
| `errorMessage` | String | `'Impossible...'` | Message erreur personnalisé |
| `showNoMessageInfo` | Boolean | `true` | Afficher infos/erreurs |
| `bannerClass` | String | `''` | Classe CSS bannière |
| `inlineClass` | String | `''` | Classe CSS message inline |
| `inlineContainerClass` | String | `''` | Classe CSS conteneur inline |
| `buttonClass` | String | `''` | Classe CSS boutons |
| `reopenButtonClass` | String | `''` | Classe CSS bouton réouverture |
```

---

## 🔑 Mots-Clés Magiques & Patterns

### Pour CSS Customization
```
❌ Éviter: "le composant doit être stylable"
✅ Utiliser: "props de classes CSS pour injection depuis le parent"
✅ Pattern: "bannerClass, buttonClass, containerClass props (String)"
```

### Pour Modes d'Affichage
```
❌ Éviter: "bannière" (ambigu)
✅ Utiliser: "mode 'banner' fixed (top/bottom) ET mode 'inline' dans le flow"
✅ Pattern: "prop displayMode avec validator: ['banner', 'inline']"
```

### Pour Slots
```
❌ Éviter: "contenu personnalisable"
✅ Utiliser: "slot Vue nommé 'button-content' pour injection"
✅ Pattern: "<slot name='X'>contenu par défaut</slot>"
```

### Pour Gestion Erreurs
```
❌ Éviter: "gérer les erreurs"
✅ Utiliser: "404 et fichiers vides = cas normaux, PAS d'erreur affichée"
✅ Pattern: "if (status === 404) return {data: [], error: false}"
```

### Pour Persistance Optimisée
```
❌ Éviter: "sauvegarder l'état"
✅ Utiliser: "localStorage avec UNIQUEMENT dernier ID lu (pas Set/Array)"
✅ Pattern: "localStorage.setItem('key', lastId) // string, pas JSON"
```

### Pour Types de Boutons
```
❌ Éviter: "bouton flottant", "bouton de réouverture"
✅ Utiliser: "bouton inline simple" ou "bouton fixed floating"
✅ Pattern: "button.reopen-btn (inline)" vs "button.floating (fixed)"
```

### Pour Props Booléens
```
❌ Dans usage: showInfo="false" (passe string!)
✅ Dans usage: :showInfo="false" (passe boolean)
✅ Dans doc: "ATTENTION: utiliser : pour binding booléens"
```

---

## 💡 Pièges Courants à Éviter

### 1. Ambiguïté de vocabulaire

| Terme ambigu | Préciser |
|--------------|----------|
| "Bannière" | "Fixed banner top/bottom" OU "Inline message bar" |
| "Popup" | "Modal overlay" OU "Tooltip" OU "Toast notification" |
| "Bouton" | "Inline button" OU "Fixed floating button" |
| "Personnalisable" | "Via props" OU "Via slots" OU "Via classes CSS" |

### 2. Gestion erreurs incomplète

**Toujours spécifier:**
- Quels cas sont "normaux" (404, vide, etc.)
- Quels cas sont "erreurs" (parse, network, etc.)
- Comportement pour chaque cas (afficher message / silent fail / retry)

### 3. Oubli de l'extensibilité

**Checklist:**
- [ ] Props de classes CSS pour tous les éléments stylables
- [ ] Slots pour tous les contenus textuels
- [ ] Props Boolean pour activer/désactiver features
- [ ] Events émis pour hooks parents

### 4. Persistance non optimisée

```javascript
// ❌ Mauvais: stocke tout
localStorage.setItem('read', JSON.stringify([...allReadIds]))

// ✅ Bon: stocke minimum
localStorage.setItem('lastRead', lastId)
```

### 5. Syntaxe Vue oubliée

```vue
<!-- ❌ Props booléens sans binding -->
<Component showInfo="false" />  <!-- passe string "false" (truthy!) -->

<!-- ✅ Props booléens avec binding -->
<Component :showInfo="false" />  <!-- passe boolean false -->
```

---

## 🚀 Version Ultra-Concise (TL;DR)

### One-liner à ajouter à TOUTES vos consignes composants:

> **"Prévoir props de classes CSS pour customisation parent, slots Vue pour contenus, gestion 404/vide comme cas normaux (pas erreurs), et si modes d'affichage multiples préciser fixed/inline/modal explicitement."**

### Checklist minimale (3 questions):

1. **Personnalisation** : Props classes CSS ? Quels slots ?
2. **Erreurs** : Qu'est-ce qui est "normal" vs "erreur" ?
3. **Affichage** : Fixed / Inline / Modal / autre ? (préciser)

---

## 📚 Exemples de Consignes (Bon vs Mauvais)

### Exemple 1: Modal de Confirmation

#### ❌ Consignes vagues
```
Créer une modale de confirmation.
Avec un bouton OK et Annuler.
Personnalisable.
```

#### ✅ Consignes précises
```
Composant Modal de Confirmation - Vue3 Composition API

AFFICHAGE:
- Overlay sombre (80% opacity) avec modale centrée
- Props classes CSS: overlayClass, modalClass, buttonClass
- Slot 'content' pour message, slot 'actions' pour boutons custom

INTERACTIONS:
- Emit 'confirm' au clic OK
- Emit 'cancel' au clic Annuler ou ESC ou clic overlay
- Prop closeOnOverlayClick (Boolean, default: true)

ACCESSIBILITÉ:
- Focus trap dans la modale
- ESC pour fermer
- ARIA attributes (role="dialog", aria-modal="true")
```

### Exemple 2: Liste de Notifications

#### ❌ Consignes vagues
```
Afficher des notifications en haut à droite.
Elles disparaissent après quelques secondes.
Types: succès, erreur, info.
```

#### ✅ Consignes précises
```
Composant NotificationStack - Vue3 Composition API

AFFICHAGE:
- Position fixed top-right (20px margin)
- Stack vertical (nouvelles en haut)
- 4 types via prop 'type': 'success', 'error', 'warning', 'info'
- Props classes CSS par type: successClass, errorClass, warningClass, infoClass

COMPORTEMENT:
- Auto-dismiss après prop 'duration' (Number, default: 5000ms)
- duration=0 : pas d'auto-dismiss
- Bouton close (×) sur chaque notification
- Max prop 'maxStack' notifications (Number, default: 5), FIFO

DONNÉES:
- Prop 'notifications' (Array): [{id, type, message, duration?}]
- Emit 'dismiss' avec notification.id

ANIMATIONS:
- Slide-in de la droite (200ms)
- Fade-out au dismiss (300ms)
```

---

## 🎨 Template Props Table

Copiez-collez et remplissez:

```markdown
| Prop | Type | Défaut | Description |
|------|------|--------|-------------|
| `primaryData` | String/Array/Object | *requis* | [Description] |
| `mode` | String | `'default'` | `'default'`, `'compact'`, `'expanded'` |
| `showInfo` | Boolean | `true` | Afficher informations supplémentaires |
| `customClass` | String | `''` | Classe CSS personnalisée pour [élément] |
| `onAction` | Function | `null` | Callback appelé lors de [action] |
```

---

## 🎯 Template Slots Table

```markdown
| Slot | Description | Contenu par défaut | Props du slot |
|------|-------------|-------------------|---------------|
| `header` | En-tête du composant | `<h2>Titre</h2>` | `{ title }` |
| `content` | Contenu principal | Message par défaut | `{ data, index }` |
| `footer` | Pied de page | Boutons action | `{ actions }` |
```

---

## 📖 Ressources Utiles

### Patterns Vue3 recommandés

- **Props validation**: Toujours utiliser `validator` pour String à valeurs limitées
- **Props Boolean**: Documenter syntaxe `:propName="false"` avec `:`
- **CSS Scoping**: `<style scoped>` + props de classes pour override
- **Slots**: Toujours fournir contenu par défaut
- **Events**: Nommer en kebab-case: `@user-action` pas `@userAction`

### Anti-patterns à éviter

- ❌ Styles inline via props (utiliser classes)
- ❌ Props de type `any` (toujours typer)
- ❌ Mutations de props (utiliser `computed` ou `ref` local)
- ❌ localStorage sans try/catch (peut être disabled)
- ❌ Fetch sans timeout (risque hang)

---

## ✅ Validation Finale

Avant de lancer le développement, vérifier:

- [ ] Tous les modes d'affichage sont explicitement listés
- [ ] Tous les éléments stylables ont une prop de classe CSS
- [ ] Tous les textes affichés ont un slot OU prop de customisation
- [ ] Les cas "normaux" vs "erreurs" sont distingués
- [ ] La persistance (si applicable) est optimisée
- [ ] Les props booléens sont documentés avec syntaxe `:`
- [ ] Un exemple d'usage complet est fourni
- [ ] Les dépendances externes sont listées (marked, etc.)

---

**Dernière mise à jour**: 2025-12-25
**Basé sur**: Retour d'expérience composant MessageBanner
