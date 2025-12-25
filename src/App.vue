<template>
  <div class="app-container">
    <!-- Composant de bannière de messages -->
    <MessageBanner
      :primary-url="primaryUrl"
      :secondary-url="secondaryUrl"
      :enable-markdown="true"
      :full-markdown="false"
      :max-history="10"
      :position="bannerPosition"
    />

    <!-- Contenu de l'application -->
    <main class="main-content">
      <div class="content-wrapper">
        <h1>Message Banner - Vue3 PWA</h1>

        <div class="demo-section">
          <h2>Démonstration du composant</h2>
          <p>
            Cette application démontre l'utilisation du composant <code>MessageBanner</code>
            dans une PWA Vue3.
          </p>

          <div class="features">
            <div class="feature-card">
              <div class="feature-icon">📱</div>
              <h3>Responsive</h3>
              <p>Optimisé pour mobile et desktop</p>
            </div>

            <div class="feature-card">
              <div class="feature-icon">💾</div>
              <h3>Persistance</h3>
              <p>Les messages lus sont mémorisés</p>
            </div>

            <div class="feature-card">
              <div class="feature-icon">🔄</div>
              <h3>Fallback</h3>
              <p>Système de secours automatique</p>
            </div>

            <div class="feature-card">
              <div class="feature-icon">📋</div>
              <h3>Historique</h3>
              <p>Consultation des messages précédents</p>
            </div>

            <div class="feature-card">
              <div class="feature-icon">🎨</div>
              <h3>Markdown</h3>
              <p>Support du formatage simple ou complet</p>
            </div>

            <div class="feature-card">
              <div class="feature-icon">⏰</div>
              <h3>Expiration</h3>
              <p>Gestion automatique des dates de validité</p>
            </div>
          </div>
        </div>

        <div class="demo-section">
          <h2>Configuration</h2>

          <div class="config-panel">
            <div class="config-item">
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
          <h2>Format des messages JSON</h2>
          <pre class="code-block"><code>{
  "messages": [
    {
      "id": "msg-001",
      "datetime": "2025-12-20T10:00:00Z",
      "expiryDate": "2026-01-20T23:59:59Z",
      "content": "Votre message ici avec **formatage** et [liens](https://example.com)"
    }
  ]
}</code></pre>
        </div>

        <div class="demo-section">
          <h2>Utilisation du composant</h2>
          <pre class="code-block"><code>&lt;MessageBanner
  primary-url="/messages-primary.json"
  secondary-url="/messages-secondary.json"
  :enable-markdown="true"
  :full-markdown="false"
  :max-history="10"
  position="top"
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
                <td><code>position</code></td>
                <td>String</td>
                <td>'top'</td>
                <td>Position de la bannière : 'top' ou 'bottom'</td>
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
    const bannerPosition = ref('top')
    const primaryUrl = ref('/messages-primary.json')
    const secondaryUrl = ref('/messages-secondary.json')

    return {
      bannerPosition,
      primaryUrl,
      secondaryUrl
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

.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.feature-card {
  text-align: center;
  padding: 1.5rem;
  background: #f8f9fa;
  border-radius: 8px;
  transition: transform 0.2s;

  &:hover {
    transform: translateY(-4px);
  }
}

.feature-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.feature-card h3 {
  color: #667eea;
  margin-bottom: 0.5rem;
  font-size: 1.2rem;
}

.feature-card p {
  color: #666;
  font-size: 0.9rem;
  margin: 0;
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

  label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-weight: normal;
    margin: 0;
    cursor: pointer;
  }

  input[type="radio"] {
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
