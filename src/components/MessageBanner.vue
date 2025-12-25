<template>
  <div v-if="shouldShowBanner" :class="['message-banner', `position-${position}`]">
    <!-- Bannière principale -->
    <div class="banner-content">
      <div class="message-text" v-html="formattedMessage"></div>
      <div class="banner-actions">
        <button
          v-if="hasHistory"
          @click="showHistoryModal = true"
          class="btn-secondary"
          title="Voir l'historique"
        >
          📋
        </button>
        <button @click="markAsRead" class="btn-primary">
          ✓ Lu
        </button>
        <button @click="closeBanner" class="btn-close">
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

  <!-- Bouton pour rouvrir le dernier message -->
  <div
    v-if="!shouldShowBanner && hasLastMessage"
    :class="[
      'reopen-button',
      `position-${position}`,
      { 'floating': floatingButton }
    ]"
  >
    <button @click="reopenBanner" class="reopen-btn" title="Afficher le dernier message">
      <span class="icon">💬</span>
      <span class="text">Message disponible</span>
    </button>
  </div>
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
    floatingButton: {
      type: Boolean,
      default: false
    },
    errorMessage: {
      type: String,
      default: 'Impossible de charger les messages. Veuillez réessayer plus tard.'
    },
    showNoMessageInfo: {
      type: Boolean,
      default: true
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

      try {
        // Essayer l'URL primaire
        const response = await fetch(props.primaryUrl)
        if (response.ok) {
          try {
            const text = await response.text()
            // Gérer le cas du fichier vide
            if (!text || text.trim() === '') {
              return { messages: [], error: false }
            }
            const data = JSON.parse(text)
            return { messages: data.messages || [], error: false }
          } catch (parseError) {
            console.error('Erreur de parsing JSON (URL primaire):', parseError)
            throw new Error('Invalid JSON in primary URL')
          }
        }
        throw new Error('Primary URL failed')
      } catch (error) {
        console.warn('Échec de l\'URL primaire, tentative avec l\'URL secondaire:', error.message)
        try {
          // Fallback sur l'URL secondaire
          const response = await fetch(props.secondaryUrl)
          if (response.ok) {
            try {
              const text = await response.text()
              // Gérer le cas du fichier vide
              if (!text || text.trim() === '') {
                return { messages: [], error: false }
              }
              const data = JSON.parse(text)
              return { messages: data.messages || [], error: false }
            } catch (parseError) {
              console.error('Erreur de parsing JSON (URL secondaire):', parseError)
              throw new Error('Invalid JSON in secondary URL')
            }
          }
          throw new Error('Secondary URL failed')
        } catch (secondaryError) {
          console.error('Impossible de récupérer les messages:', secondaryError.message)
          fetchError.value = true
          return { messages: [], error: true }
        }
      }
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
      // Si aucun message n'a été lu, retourner le premier (le plus récent)
      if (!lastReadMessageId.value) {
        return msgs[0] || null
      }

      // Sinon, retourner le premier message qui n'est pas le dernier lu
      // (on affiche uniquement les messages plus récents que le dernier lu)
      const lastReadIndex = msgs.findIndex(msg => msg.id === lastReadMessageId.value)

      // Si le dernier message lu n'est plus dans la liste ou qu'il y a de nouveaux messages
      if (lastReadIndex === -1 || lastReadIndex > 0) {
        return msgs[0]
      }

      return null
    }

    // Initialiser les messages
    const initMessages = async () => {
      const result = await fetchMessages()
      hasEverFetched.value = true

      // Si erreur de fetch, afficher le message d'erreur uniquement s'il y a vraiment une erreur
      if (result.error && result.messages.length === 0) {
        // Créer un message d'erreur temporaire
        currentMessage.value = {
          id: 'error-message',
          datetime: new Date().toISOString(),
          expiryDate: new Date(Date.now() + 3600000).toISOString(), // 1 heure
          content: props.errorMessage,
          isError: true
        }
        showBanner.value = true
        return
      }

      messages.value = filterValidMessages(result.messages)

      // Si aucun message disponible et que l'option est activée, afficher un message d'info
      if (messages.value.length === 0 && props.showNoMessageInfo) {
        currentMessage.value = {
          id: 'no-message-info',
          datetime: new Date().toISOString(),
          expiryDate: new Date(Date.now() + 3600000).toISOString(), // 1 heure
          content: 'Aucun message disponible pour le moment.',
          isInfo: true
        }
        showBanner.value = true
        return
      }

      // Si aucun message et l'option est désactivée, ne rien afficher
      if (messages.value.length === 0) {
        return
      }

      const unreadMessage = findUnreadMessage(messages.value)
      if (unreadMessage) {
        currentMessage.value = unreadMessage
        showBanner.value = true
      }
    }

    // Marquer un message comme lu
    const markAsRead = () => {
      if (currentMessage.value && !currentMessage.value.isError && !currentMessage.value.isInfo) {
        // Sauvegarder uniquement le dernier message lu
        saveLastReadMessage(currentMessage.value.id)
        showBanner.value = false

        // Chercher le prochain message non lu
        const nextUnread = findUnreadMessage(messages.value)
        if (nextUnread) {
          setTimeout(() => {
            currentMessage.value = nextUnread
            showBanner.value = true
          }, 300)
        }
      } else if (currentMessage.value && (currentMessage.value.isError || currentMessage.value.isInfo)) {
        // Pour un message d'erreur ou d'info, on ferme juste la bannière
        showBanner.value = false
      }
    }

    // Fermer la bannière temporairement
    const closeBanner = () => {
      showBanner.value = false
    }

    // Rouvrir la bannière
    const reopenBanner = () => {
      if (messages.value.length > 0) {
        currentMessage.value = messages.value[0]
        showBanner.value = true
      } else if (fetchError.value) {
        // Si erreur de fetch, réafficher le message d'erreur
        initMessages()
      } else if (hasEverFetched.value && props.showNoMessageInfo) {
        // Si pas de messages mais fetch réussi, afficher le message d'info
        currentMessage.value = {
          id: 'no-message-info',
          datetime: new Date().toISOString(),
          expiryDate: new Date(Date.now() + 3600000).toISOString(),
          content: 'Aucun message disponible pour le moment.',
          isInfo: true
        }
        showBanner.value = true
      } else {
        // Sinon, réessayer de fetch les messages
        initMessages()
      }
    }

    // Vérifier si un message a été lu
    const isMessageRead = (messageId) => {
      // Un message est considéré comme lu s'il est plus ancien ou égal au dernier message lu
      if (!lastReadMessageId.value) return false

      const messageIndex = messages.value.findIndex(msg => msg.id === messageId)
      const lastReadIndex = messages.value.findIndex(msg => msg.id === lastReadMessageId.value)

      // Si on trouve les deux, comparer les index (les messages sont triés du plus récent au plus ancien)
      if (messageIndex !== -1 && lastReadIndex !== -1) {
        return messageIndex >= lastReadIndex
      }

      // Si le message n'est pas trouvé ou le dernier lu n'est pas trouvé, considérer comme lu
      return messageIndex === -1
    }

    // Formater le contenu du message
    const formatMessageContent = (content) => {
      if (!content) return ''

      if (props.enableMarkdown) {
        if (props.fullMarkdown) {
          // Markdown complet
          return marked(content)
        } else {
          // Markdown simple : liens, gras, italique
          let formatted = content
          // Liens markdown [texte](url)
          formatted = formatted.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2" target="_blank" rel="noopener noreferrer">$1</a>')
          // Gras **texte**
          formatted = formatted.replace(/\*\*([^*]+)\*\*/g, '<strong>$1</strong>')
          // Italique *texte*
          formatted = formatted.replace(/\*([^*]+)\*/g, '<em>$1</em>')
          return formatted
        }
      } else {
        // Pas de markdown, mais détecter les URLs
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

.banner-actions {
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
  animation: fadeIn 0.2s ease-out;
}

@keyframes fadeIn {
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

// Bouton de réouverture
.reopen-button {
  position: fixed;
  left: 0;
  right: 0;
  z-index: 999;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);

  &.position-top {
    top: 0;
  }

  &.position-bottom {
    bottom: 0;
  }

  // Mode bouton normal (par défaut) - barre horizontale
  .reopen-btn {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    padding: 0.75rem 1rem;
    background: transparent;
    color: white;
    font-weight: 500;
    font-size: 0.9rem;

    .icon {
      font-size: 1.1rem;
    }

    .text {
      @media (max-width: 480px) {
        display: none;
      }
    }

    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }
  }

  // Mode bouton flottant (optionnel)
  &.floating {
    position: fixed;
    left: auto;
    right: 1rem;
    width: auto;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 50px;
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);

    &.position-top {
      top: 1rem;
    }

    &.position-bottom {
      bottom: 1rem;
    }

    .reopen-btn {
      width: auto;
      padding: 0.75rem 1rem;
      border-radius: 50px;

      .text {
        display: none;
      }

      .icon {
        font-size: 1.25rem;
      }
    }

    &:hover {
      box-shadow: 0 6px 16px rgba(102, 126, 234, 0.6);
    }
  }
}
</style>
