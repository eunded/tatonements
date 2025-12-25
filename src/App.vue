<template>
  <div class="app-container">
    <!-- Exemple mode bannière -->
    <MessageBanner
      v-if="displayMode === 'banner'"
      :primary-url="primaryUrl"
      :secondary-url="secondaryUrl"
      :enable-markdown="true"
      :full-markdown="false"
      :max-history="10"
      :position="bannerPosition"
      display-mode="banner"
      :auto-open="autoOpen"
      :auto-open-when-empty="autoOpenWhenEmpty"
      error-message="⚠️ Impossible de charger les messages. Veuillez vérifier votre connexion."
      :banner-class="customBannerClass"
      :button-class="customButtonClass"
      :reopen-button-class="customReopenButtonClass"
    >
      <template #reopen-button-content>
        <span>💬</span>
        <span v-if="showButtonText">Messages</span>
      </template>
    </MessageBanner>

    <!-- Contenu de l'application -->
    <main class="main-content">
      <div class="content-wrapper">
        <h1>Message Banner - Vue3 PWA</h1>

        <div class="demo-section">
          <h2>Démonstration des modes d'affichage</h2>

          <!-- Exemple mode inline -->
          <div class="inline-demo">
            <h3>Mode Inline</h3>
            <p>Le message s'affiche directement dans la page :</p>

            <MessageBanner
              v-if="displayMode === 'inline'"
              :primary-url="primaryUrl"
              :secondary-url="secondaryUrl"
              :enable-markdown="true"
              :full-markdown="false"
              :max-history="10"
              display-mode="inline"
              :auto-open="autoOpen"
              :auto-open-when-empty="autoOpenWhenEmpty"
              error-message="⚠️ Impossible de charger les messages."
              :inline-class="customInlineClass"
              :button-class="customButtonClass"
              :reopen-button-class="customReopenButtonClass"
            >
              <template #reopen-button-content>
                💬 Voir les messages
              </template>
            </MessageBanner>
          </div>
        </div>

        <div class="demo-section">
          <h2>Configuration</h2>

          <div class="config-panel">
            <div class="config-item">
              <label>Mode d'affichage :</label>
              <div class="radio-group">
                <label>
                  <input
                    type="radio"
                    value="banner"
                    v-model="displayMode"
                  />
                  Bannière fixe (top/bottom)
                </label>
                <label>
                  <input
                    type="radio"
                    value="inline"
                    v-model="displayMode"
                  />
                  Inline (dans la page)
                </label>
              </div>
            </div>

            <div class="config-item" v-if="displayMode === 'banner'">
              <label>Position de la bannière :</label>
              <div class="radio-group">
                <label>
                  <input
                    type="radio"
                    value="top"
                    v-model="bannerPosition"
                  />
                  En haut
                </label>
                <label>
                  <input
                    type="radio"
                    value="bottom"
                    v-model="bannerPosition"
                  />
                  En bas
                </label>
              </div>
            </div>

            <div class="config-item">
              <label>
                <input
                  type="checkbox"
                  v-model="showButtonText"
                />
                Afficher le texte du bouton de réouverture
              </label>
            </div>

            <div class="config-item">
              <label>URL primaire :</label>
              <input
                type="text"
                v-model="primaryUrl"
                class="url-input"
                placeholder="/messages-primary.json"
              />
            </div>

            <div class="config-item">
              <label>URL secondaire (fallback) :</label>
              <input
                type="text"
                v-model="secondaryUrl"
                class="url-input"
                placeholder="/messages-secondary.json"
              />
            </div>
          </div>
        </div>

        <div class="demo-section">
          <h2>Personnalisation CSS</h2>
          <p>Exemple de personnalisation avec des classes CSS depuis le parent :</p>

          <pre class="code-block"><code>&lt;MessageBanner
  display-mode="banner"
  banner-class="my-custom-banner"
  button-class="my-custom-button"
  reopen-button-class="my-custom-reopen-btn"
&gt;
  &lt;template #reopen-button-content&gt;
    💬 Voir les messages
  &lt;/template&gt;
&lt;/MessageBanner&gt;

&lt;style&gt;
.my-custom-banner {
  background: linear-gradient(to right, #ff6b6b, #ee5a6f);
}

.my-custom-button {
  border-radius: 20px;
  font-weight: bold;
}

.my-custom-reopen-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
  border-radius: 50px;
  padding: 1rem 2rem;
}
&lt;/style&gt;</code></pre>
        </div>

        <div class="demo-section">
          <h2>Utilisation du composant</h2>

          <h3>Mode Bannière</h3>
          <pre class="code-block"><code>&lt;MessageBanner
  primary-url="/messages-primary.json"
  secondary-url="/messages-secondary.json"
  :enable-markdown="true"
  display-mode="banner"
  position="top"
  error-message="Impossible de charger les messages."
/&gt;</code></pre>

          <h3>Mode Inline</h3>
          <pre class="code-block"><code>&lt;MessageBanner
  primary-url="/messages-primary.json"
  secondary-url="/messages-secondary.json"
  :enable-markdown="true"
  display-mode="inline"
  error-message="Impossible de charger les messages."
/&gt;</code></pre>
        </div>

        <div class="demo-section">
          <h2>Props disponibles</h2>
          <table class="props-table">
            <thead>
              <tr>
                <th>Prop</th>
                <th>Type</th>
                <th>Défaut</th>
                <th>Description</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code>primaryUrl</code></td>
                <td>String</td>
                <td>-</td>
                <td>URL principale pour récupérer les messages</td>
              </tr>
              <tr>
                <td><code>secondaryUrl</code></td>
                <td>String</td>
                <td>-</td>
                <td>URL de secours si la primaire échoue</td>
              </tr>
              <tr>
                <td><code>displayMode</code></td>
                <td>String</td>
                <td>'banner'</td>
                <td>Mode d'affichage : 'banner' (fixe) ou 'inline' (dans la page)</td>
              </tr>
              <tr>
                <td><code>position</code></td>
                <td>String</td>
                <td>'top'</td>
                <td>Position de la bannière (si mode=banner) : 'top' ou 'bottom'</td>
              </tr>
              <tr>
                <td><code>enableMarkdown</code></td>
                <td>Boolean</td>
                <td>false</td>
                <td>Activer le support Markdown</td>
              </tr>
              <tr>
                <td><code>fullMarkdown</code></td>
                <td>Boolean</td>
                <td>false</td>
                <td>Markdown complet (sinon simple : liens, gras, italique)</td>
              </tr>
              <tr>
                <td><code>maxHistory</code></td>
                <td>Number</td>
                <td>10</td>
                <td>Nombre de messages dans l'historique</td>
              </tr>
              <tr>
                <td><code>errorMessage</code></td>
                <td>String</td>
                <td>'Impossible de charger...'</td>
                <td>Message affiché si les deux URLs échouent</td>
              </tr>
              <tr>
                <td><code>autoOpen</code></td>
                <td>Boolean</td>
                <td>true</td>
                <td>Ouvrir automatiquement la bannière au montage du composant</td>
              </tr>
              <tr>
                <td><code>autoOpenWhenEmpty</code></td>
                <td>Boolean</td>
                <td>true</td>
                <td>Si autoOpen=true et aucun message, afficher quand même un placeholder</td>
              </tr>
              <tr>
                <td><code>bannerClass</code></td>
                <td>String</td>
                <td>''</td>
                <td>Classe CSS personnalisée pour la bannière</td>
              </tr>
              <tr>
                <td><code>inlineClass</code></td>
                <td>String</td>
                <td>''</td>
                <td>Classe CSS personnalisée pour le message inline</td>
              </tr>
              <tr>
                <td><code>inlineContainerClass</code></td>
                <td>String</td>
                <td>''</td>
                <td>Classe CSS personnalisée pour le conteneur inline</td>
              </tr>
              <tr>
                <td><code>buttonClass</code></td>
                <td>String</td>
                <td>''</td>
                <td>Classe CSS personnalisée pour les boutons (Lu, Historique, Fermer)</td>
              </tr>
              <tr>
                <td><code>reopenButtonClass</code></td>
                <td>String</td>
                <td>''</td>
                <td>Classe CSS personnalisée pour le bouton de réouverture</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="demo-section">
          <h2>Slots disponibles</h2>
          <table class="props-table">
            <thead>
              <tr>
                <th>Slot</th>
                <th>Description</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code>reopen-button-content</code></td>
                <td>Contenu du bouton de réouverture. Par défaut : "💬 Messages"</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { ref } from 'vue'
import MessageBanner from './components/MessageBanner.vue'

export default {
  name: 'App',
  components: {
    MessageBanner
  },
  setup() {
    const displayMode = ref('banner')
    const bannerPosition = ref('top')
    const showButtonText = ref(true)
    const primaryUrl = ref('/messages-primary.json')
    const secondaryUrl = ref('/messages-secondary.json')
    const autoOpen = ref(true)
    const autoOpenWhenEmpty = ref(true)

    // Classes personnalisées (vides par défaut, utilisateur peut les remplir)
    const customBannerClass = ref('')
    const customInlineClass = ref('')
    const customButtonClass = ref('')
    const customReopenButtonClass = ref('')

    return {
      displayMode,
      bannerPosition,
      showButtonText,
      primaryUrl,
      secondaryUrl,
      autoOpen,
      autoOpenWhenEmpty,
      customBannerClass,
      customInlineClass,
      customButtonClass,
      customReopenButtonClass
    }
  }
}
</script>

<style lang="scss" scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.main-content {
  padding: 2rem 1rem;
  padding-top: 5rem; // Espace pour la bannière en haut

  @media (max-width: 768px) {
    padding: 1rem;
    padding-top: 6rem;
  }
}

.content-wrapper {
  max-width: 900px;
  margin: 0 auto;
}

h1 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 2rem;
  font-size: 2.5rem;

  @media (max-width: 768px) {
    font-size: 1.8rem;
  }
}

h2 {
  color: #34495e;
  margin-bottom: 1rem;
  font-size: 1.5rem;
  border-bottom: 2px solid #667eea;
  padding-bottom: 0.5rem;
}

h3 {
  color: #667eea;
  margin-top: 1.5rem;
  margin-bottom: 0.75rem;
  font-size: 1.2rem;
}

.demo-section {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);

  @media (max-width: 768px) {
    padding: 1.5rem;
  }
}

.inline-demo {
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  margin-top: 1rem;
}

.config-panel {
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
}

.config-item {
  margin-bottom: 1.5rem;

  &:last-child {
    margin-bottom: 0;
  }

  label {
    display: block;
    color: #34495e;
    font-weight: 600;
    margin-bottom: 0.5rem;
  }
}

.radio-group {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap;

  label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-weight: normal;
    margin: 0;
    cursor: pointer;
  }

  input[type="radio"],
  input[type="checkbox"] {
    cursor: pointer;
  }
}

.url-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  font-family: monospace;
  font-size: 0.9rem;
  transition: border-color 0.2s;

  &:focus {
    outline: none;
    border-color: #667eea;
  }
}

.code-block {
  background: #2d3748;
  color: #e2e8f0;
  padding: 1.5rem;
  border-radius: 8px;
  overflow-x: auto;
  margin: 1rem 0;

  code {
    font-family: 'Courier New', monospace;
    font-size: 0.9rem;
    line-height: 1.6;
  }
}

.props-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;

  th, td {
    padding: 0.75rem;
    text-align: left;
    border-bottom: 1px solid #e0e0e0;
  }

  th {
    background: #f8f9fa;
    font-weight: 600;
    color: #34495e;
  }

  code {
    background: #f0f0f0;
    padding: 0.2rem 0.4rem;
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.85rem;
    color: #667eea;
  }

  tr:hover {
    background: #f8f9fa;
  }
}

p {
  color: #555;
  line-height: 1.6;
}
</style>
