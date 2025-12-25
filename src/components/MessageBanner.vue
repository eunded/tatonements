<template>
  <!-- Mode bannière fixe -->
  <div v-if="displayMode === 'banner' && shouldShowBanner"
       :class="['message-banner', `position-${position}`, bannerClass]">
    <div class="banner-content">
      <div class="message-text" v-html="formattedMessage"></div>
      <div class="banner-actions">
        <button
          v-if="hasHistory"
          @click="showHistoryModal = true"
          :class="['btn-secondary', buttonClass]"
          title="Voir l'historique"
        >
          📋
        </button>
        <button @click="markAsRead" :class="['btn-primary', buttonClass]">
          ✓ Lu
        </button>
        <button @click="closeBanner" :class="['btn-close', buttonClass]">
          ✕
        </button>
      </div>
    </div>

    <!-- Modale d'historique -->
    <div v-if="showHistoryModal" class="modal-overlay" @click.self="showHistoryModal = false">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Historique des messages</h3>
          <button @click="showHistoryModal = false" class="btn-close">✕</button>
        </div>
        <div class="modal-body">
          <div
            v-for="msg in historyMessages"
            :key="msg.id"
            class="history-item"
            :class="{ 'read': isMessageRead(msg.id) }"
          >
            <div class="history-date">
              {{ formatDate(msg.datetime) }}
              <span v-if="isMessageRead(msg.id)" class="read-badge">Lu</span>
            </div>
            <div class="history-content" v-html="formatMessageContent(msg.content)"></div>
          </div>
          <div v-if="historyMessages.length === 0" class="no-history">
            Aucun message dans l'historique
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Mode inline -->
  <div v-if="displayMode === 'inline'" :class="['message-inline-container', inlineContainerClass]">
    <div v-if="shouldShowBanner" :class="['message-inline', inlineClass]">
      <div class="message-text" v-html="formattedMessage"></div>
      <div class="message-actions">
        <button
          v-if="hasHistory"
          @click="showHistoryModal = true"
          :class="['btn-secondary', buttonClass]"
          title="Voir l'historique"
        >
          📋
        </button>
        <button @click="markAsRead" :class="['btn-primary', buttonClass]">
          ✓ Lu
        </button>
        <button @click="closeBanner" :class="['btn-close', buttonClass]">
          ✕
        </button>
      </div>
    </div>

    <!-- Modale d'historique pour inline -->
    <div v-if="showHistoryModal" class="modal-overlay" @click.self="showHistoryModal = false">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Historique des messages</h3>
          <button @click="showHistoryModal = false" class="btn-close">✕</button>
        </div>
        <div class="modal-body">
          <div
            v-for="msg in historyMessages"
            :key="msg.id"
            class="history-item"
            :class="{ 'read': isMessageRead(msg.id) }"
          >
            <div class="history-date">
              {{ formatDate(msg.datetime) }}
              <span v-if="isMessageRead(msg.id)" class="read-badge">Lu</span>
            </div>
            <div class="history-content" v-html="formatMessageContent(msg.content)"></div>
          </div>
          <div v-if="historyMessages.length === 0" class="no-history">
            Aucun message dans l'historique
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Bouton de réouverture (simple bouton) -->
  <button
    v-if="!shouldShowBanner && hasLastMessage"
    @click="reopenBanner"
    :class="['reopen-button', reopenButtonClass]"
    title="Afficher le dernier message"
  >
    <slot name="reopen-button-content">
      💬 Messages
    </slot>
  </button>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { marked } from 'marked'

export default {
  name: 'MessageBanner',
  props: {
    primaryUrl: {
      type: String,
      required: true
    },
    secondaryUrl: {
      type: String,
      required: true
    },
    enableMarkdown: {
      type: Boolean,
      default: false
    },
    fullMarkdown: {
      type: Boolean,
      default: false
    },
    maxHistory: {
      type: Number,
      default: 10
    },
    position: {
      type: String,
      default: 'top',
      validator: (value) => ['top', 'bottom'].includes(value)
    },
    displayMode: {
      type: String,
      default: 'banner',
      validator: (value) => ['banner', 'inline'].includes(value)
    },
    errorMessage: {
      type: String,
      default: 'Impossible de charger les messages. Veuillez réessayer plus tard.'
    },
    autoOpen: {
      type: Boolean,
      default: true
    },
    autoOpenWhenEmpty: {
      type: Boolean,
      default: true
    },
    // Props pour personnaliser les classes CSS
    bannerClass: {
      type: String,
      default: ''
    },
    inlineClass: {
      type: String,
      default: ''
    },
    inlineContainerClass: {
      type: String,
      default: ''
    },
    buttonClass: {
      type: String,
      default: ''
    },
    reopenButtonClass: {
      type: String,
      default: ''
    }
  },
  setup(props) {
    const messages = ref([])
    const currentMessage = ref(null)
    const showBanner = ref(false)
    const showHistoryModal = ref(false)
    const lastReadMessageId = ref(null)
    const fetchError = ref(false)
    const hasEverFetched = ref(false)

    // Clé pour localStorage - ne garde que le dernier message lu
    const STORAGE_KEY = 'message-banner-last-read'

    // Charger le dernier message lu depuis localStorage
    const loadLastReadMessage = () => {
      try {
        const stored = localStorage.getItem(STORAGE_KEY)
        if (stored) {
          lastReadMessageId.value = stored
        }
      } catch (error) {
        console.error('Erreur lors du chargement du dernier message lu:', error)
      }
    }

    // Sauvegarder le dernier message lu dans localStorage
    const saveLastReadMessage = (messageId) => {
      try {
        localStorage.setItem(STORAGE_KEY, messageId)
        lastReadMessageId.value = messageId
      } catch (error) {
        console.error('Erreur lors de la sauvegarde du dernier message lu:', error)
      }
    }

    // Fetch des messages avec fallback
    const fetchMessages = async () => {
      fetchError.value = false

      // Helper pour parser JSON avec gestion d'erreurs
      const parseJSON = (text, source) => {
        if (!text || text.trim() === '') {
          return { messages: [], error: false, isEmpty: true }
        }
        try {
          const data = JSON.parse(text)
          return { messages: data.messages || [], error: false, isEmpty: false }
        } catch (parseError) {
          console.error(`Erreur de parsing JSON (${source}):`, parseError.message)
          return { messages: [], error: true, parseError: true }
        }
      }

      // Helper pour fetch avec gestion complète des erreurs
      const fetchURL = async (url, source) => {
        try {
          const response = await fetch(url)

          // Fichier trouvé
          if (response.ok) {
            const text = await response.text()
            const result = parseJSON(text, source)

            if (result.isEmpty) {
              console.info(`Fichier ${source} vide, aucun message disponible`)
              return { messages: [], error: false, notFound: false }
            }

            if (result.parseError) {
              return { messages: [], error: true, notFound: false }
            }

            return { messages: result.messages, error: false, notFound: false }
          }

          // Fichier absent (404) - pas une vraie erreur
          if (response.status === 404) {
            console.info(`Fichier ${source} absent (404)`)
            return { messages: [], error: false, notFound: true }
          }

          // Autre erreur HTTP
          console.error(`Erreur HTTP ${response.status} pour ${source}`)
          return { messages: [], error: true, notFound: false }

        } catch (networkError) {
          // Erreur réseau (pas de connexion, CORS, etc.)
          console.error(`Erreur réseau pour ${source}:`, networkError.message)
          return { messages: [], error: true, notFound: false }
        }
      }

      // Essayer l'URL primaire
      const primaryResult = await fetchURL(props.primaryUrl, 'primaire')

      // Si succès avec l'URL primaire
      if (!primaryResult.notFound && primaryResult.messages.length > 0) {
        return primaryResult
      }

      // Si l'URL primaire a une vraie erreur (pas juste 404), on signale quand même
      if (primaryResult.error && !primaryResult.notFound) {
        console.warn('Problème avec URL primaire, tentative avec URL secondaire')
      }

      // Essayer l'URL secondaire
      const secondaryResult = await fetchURL(props.secondaryUrl, 'secondaire')

      // Si succès avec l'URL secondaire
      if (!secondaryResult.notFound && secondaryResult.messages.length > 0) {
        return secondaryResult
      }

      // Les deux fichiers sont absents - comportement normal, pas d'erreur
      if (primaryResult.notFound && secondaryResult.notFound) {
        console.info('Aucun fichier de messages disponible (404 sur les deux URLs)')
        return { messages: [], error: false }
      }

      // Vraie erreur sur les deux URLs (réseau, parsing, HTTP)
      if (primaryResult.error && secondaryResult.error) {
        console.error('Impossible de récupérer les messages depuis les deux URLs')
        fetchError.value = true
        return { messages: [], error: true }
      }

      // Au moins une URL fonctionne mais sans messages
      return { messages: [], error: false }
    }

    // Filtrer les messages valides (non expirés)
    const filterValidMessages = (msgs) => {
      const now = new Date()
      return msgs.filter(msg => {
        const expiryDate = new Date(msg.expiryDate)
        return expiryDate > now
      }).sort((a, b) => new Date(b.datetime) - new Date(a.datetime))
    }

    // Trouver le dernier message non lu
    const findUnreadMessage = (msgs) => {
      if (!lastReadMessageId.value) {
        return msgs[0] || null
      }

      const lastReadIndex = msgs.findIndex(msg => msg.id === lastReadMessageId.value)

      if (lastReadIndex === -1 || lastReadIndex > 0) {
        return msgs[0]
      }

      return null
    }

    // Initialiser les messages
    const initMessages = async () => {
      const result = await fetchMessages()
      hasEverFetched.value = true

      messages.value = filterValidMessages(result.messages)

      // Si autoOpen = false, ne rien ouvrir au montage
      if (!props.autoOpen) {
        showBanner.value = false
        currentMessage.value = null
        return
      }

      // autoOpen = true : vérifier si on a des messages
      if (messages.value.length > 0) {
        // Des messages existent : ouvrir avec le premier message
        currentMessage.value = messages.value[0]
        showBanner.value = true
        return
      }

      // Aucun message disponible
      // Si autoOpenWhenEmpty = true, ouvrir avec message placeholder
      if (props.autoOpenWhenEmpty) {
        // Erreur de récupération
        if (result.error) {
          currentMessage.value = {
            id: 'error-message',
            datetime: new Date().toISOString(),
            expiryDate: new Date(Date.now() + 3600000).toISOString(),
            content: props.errorMessage,
            isError: true
          }
        } else {
          // Pas d'erreur, juste pas de messages
          currentMessage.value = {
            id: 'no-message-info',
            datetime: new Date().toISOString(),
            expiryDate: new Date(Date.now() + 3600000).toISOString(),
            content: 'Aucun message disponible pour le moment.',
            isInfo: true
          }
        }
        showBanner.value = true
      } else {
        // autoOpenWhenEmpty = false : rester fermé, juste afficher le bouton
        showBanner.value = false
        currentMessage.value = null
      }
    }

    // Marquer un message comme lu
    const markAsRead = () => {
      if (currentMessage.value && !currentMessage.value.isError && !currentMessage.value.isInfo) {
        saveLastReadMessage(currentMessage.value.id)
        showBanner.value = false

        const nextUnread = findUnreadMessage(messages.value)
        if (nextUnread) {
          setTimeout(() => {
            currentMessage.value = nextUnread
            showBanner.value = true
          }, 300)
        }
      } else if (currentMessage.value && (currentMessage.value.isError || currentMessage.value.isInfo)) {
        showBanner.value = false
      }
    }

    // Fermer la bannière temporairement
    const closeBanner = () => {
      showBanner.value = false
    }

    // Rouvrir la bannière
    const reopenBanner = () => {
      // Au clic, TOUJOURS ouvrir (ignore autoOpen et autoOpenWhenEmpty)
      if (messages.value.length > 0) {
        // Des messages existent : afficher le premier
        currentMessage.value = messages.value[0]
        showBanner.value = true
      } else {
        // Aucun message : toujours afficher un placeholder
        currentMessage.value = {
          id: 'no-message-info',
          datetime: new Date().toISOString(),
          expiryDate: new Date(Date.now() + 3600000).toISOString(),
          content: 'Aucun message disponible pour le moment.',
          isInfo: true
        }
        showBanner.value = true
      }
    }

    // Vérifier si un message a été lu
    const isMessageRead = (messageId) => {
      if (!lastReadMessageId.value) return false

      const messageIndex = messages.value.findIndex(msg => msg.id === messageId)
      const lastReadIndex = messages.value.findIndex(msg => msg.id === lastReadMessageId.value)

      if (messageIndex !== -1 && lastReadIndex !== -1) {
        return messageIndex >= lastReadIndex
      }

      return messageIndex === -1
    }

    // Formater le contenu du message
    const formatMessageContent = (content) => {
      if (!content) return ''

      if (props.enableMarkdown) {
        if (props.fullMarkdown) {
          return marked(content)
        } else {
          let formatted = content
          formatted = formatted.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2" target="_blank" rel="noopener noreferrer">$1</a>')
          formatted = formatted.replace(/\*\*([^*]+)\*\*/g, '<strong>$1</strong>')
          formatted = formatted.replace(/\*([^*]+)\*/g, '<em>$1</em>')
          return formatted
        }
      } else {
        return content.replace(
          /(https?:\/\/[^\s]+)/g,
          '<a href="$1" target="_blank" rel="noopener noreferrer">$1</a>'
        )
      }
    }

    // Formater la date
    const formatDate = (dateStr) => {
      const date = new Date(dateStr)
      return date.toLocaleDateString('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
    }

    // Computed properties
    const shouldShowBanner = computed(() => showBanner.value && currentMessage.value)

    const formattedMessage = computed(() => {
      if (!currentMessage.value) return ''
      return formatMessageContent(currentMessage.value.content)
    })

    const hasLastMessage = computed(() => hasEverFetched.value || messages.value.length > 0)

    const hasHistory = computed(() => messages.value.length > 1)

    const historyMessages = computed(() => {
      return messages.value.slice(0, props.maxHistory)
    })

    // Lifecycle
    onMounted(() => {
      loadLastReadMessage()
      initMessages()
    })

    return {
      shouldShowBanner,
      formattedMessage,
      showHistoryModal,
      hasHistory,
      hasLastMessage,
      historyMessages,
      markAsRead,
      closeBanner,
      reopenBanner,
      isMessageRead,
      formatMessageContent,
      formatDate
    }
  }
}
</script>

<style lang="scss" scoped>
// Mode bannière fixe
.message-banner {
  position: fixed;
  left: 0;
  right: 0;
  z-index: 1000;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  animation: slideIn 0.3s ease-out;

  &.position-top {
    top: 0;
  }

  &.position-bottom {
    bottom: 0;
  }
}

@keyframes slideIn {
  from {
    transform: translateY(-100%);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.banner-content {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.875rem 1rem;
  max-width: 1200px;
  margin: 0 auto;

  @media (max-width: 768px) {
    flex-direction: column;
    align-items: stretch;
    gap: 0.75rem;
  }
}

// Mode inline
.message-inline-container {
  width: 100%;
}

.message-inline {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 0.3s ease-out;

  .message-text {
    margin-bottom: 1rem;
  }

  .message-actions {
    display: flex;
    gap: 0.5rem;
    justify-content: flex-end;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.message-text {
  flex: 1;
  font-size: 0.95rem;
  line-height: 1.5;

  :deep(a) {
    color: #fff;
    text-decoration: underline;
    font-weight: 500;

    &:hover {
      text-decoration: none;
    }
  }

  :deep(strong) {
    font-weight: 700;
  }

  :deep(em) {
    font-style: italic;
  }
}

.banner-actions,
.message-actions {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;

  @media (max-width: 768px) {
    justify-content: flex-end;
  }
}

button {
  border: none;
  cursor: pointer;
  font-family: inherit;
  font-size: 0.875rem;
  transition: all 0.2s;

  &:hover {
    transform: translateY(-1px);
  }

  &:active {
    transform: translateY(0);
  }
}

.btn-primary {
  background: white;
  color: #667eea;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-weight: 600;

  &:hover {
    background: #f0f0f0;
  }
}

.btn-secondary {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-weight: 600;

  &:hover {
    background: rgba(255, 255, 255, 0.3);
  }
}

.btn-close {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-weight: 600;
  font-size: 1.1rem;
  line-height: 1;

  &:hover {
    background: rgba(255, 255, 255, 0.3);
  }
}

// Modale
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 1rem;
  animation: modalFadeIn 0.2s ease-out;
}

@keyframes modalFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal-content {
  background: white;
  border-radius: 12px;
  max-width: 600px;
  width: 100%;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  animation: scaleIn 0.2s ease-out;
}

@keyframes scaleIn {
  from {
    transform: scale(0.9);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.25rem 1.5rem;
  border-bottom: 1px solid #e0e0e0;

  h3 {
    margin: 0;
    color: #333;
    font-size: 1.25rem;
  }

  .btn-close {
    background: transparent;
    color: #666;
    padding: 0.25rem 0.5rem;

    &:hover {
      background: #f0f0f0;
      color: #333;
    }
  }
}

.modal-body {
  overflow-y: auto;
  padding: 1rem 1.5rem;
  flex: 1;
}

.history-item {
  padding: 1rem;
  margin-bottom: 0.75rem;
  background: #f9f9f9;
  border-radius: 8px;
  border-left: 4px solid #667eea;

  &.read {
    opacity: 0.7;
    border-left-color: #ccc;
  }

  &:last-child {
    margin-bottom: 0;
  }
}

.history-date {
  font-size: 0.8rem;
  color: #666;
  margin-bottom: 0.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.read-badge {
  background: #4caf50;
  color: white;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
}

.history-content {
  color: #333;
  line-height: 1.6;

  :deep(a) {
    color: #667eea;
    text-decoration: none;

    &:hover {
      text-decoration: underline;
    }
  }

  :deep(strong) {
    font-weight: 700;
  }

  :deep(em) {
    font-style: italic;
  }
}

.no-history {
  text-align: center;
  color: #999;
  padding: 2rem;
  font-style: italic;
}

// Bouton de réouverture (simple bouton)
.reopen-button {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
  font-weight: 600;
  font-size: 0.95rem;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;

  &:hover {
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.5);
  }
}
</style>
