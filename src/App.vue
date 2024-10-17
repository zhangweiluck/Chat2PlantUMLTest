<template>
  <div id="app">
    <header>
      <h1>PlantUML AI Assistant</h1>
      <div class="tools">
        <button @click="copyCode" class="btn btn-primary">Copy Code</button>
        <button @click="exportJPEG" class="btn btn-secondary">Export JPEG</button>
      </div>
    </header>
    <main>
      <nav class="chat-nav">
        <button @click="newChat" class="btn btn-primary new-chat">New Chat</button>
        <ul class="chat-list">
          <li v-for="(chat, index) in chats" :key="index" @click="switchChat(index)" :class="{ active: currentChatIndex === index }">
            Chat {{ index + 1 }}
          </li>
        </ul>
      </nav>
      <div v-if="currentChat" class="chat-area">
        <div class="chat-messages">
          <div v-for="(message, index) in currentChat.messages" :key="index" :class="message.type">
            {{ message.content }}
          </div>
        </div>
        <div class="chat-input">
          <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Type your message...">
          <button @click="sendMessage" class="btn btn-primary">Send</button>
        </div>
      </div>
      <div v-if="currentChat" class="editor-preview">
        <div class="code-preview">
          <MonacoEditor
            v-model="currentChat.plantUMLCode"
            language="plantuml"
            @change="updateUMLDiagram"
          />
        </div>
        <div class="uml-diagram">
          <img :src="currentChat.umlImageUrl" alt="UML Diagram" ref="umlImage" />
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import MonacoEditor from './components/MonacoEditor.vue'
import plantumlEncoder from 'plantuml-encoder'
import axios from 'axios'

export default {
  components: {
    MonacoEditor
  },
  setup() {
    const userInput = ref('')
    const chats = ref([])
    const currentChatIndex = ref(0)

    const currentChat = computed(() => chats.value[currentChatIndex.value] || null)

    const createNewChat = () => ({
      messages: [],
      plantUMLCode: '@startuml\nclass User\nclass Order\nUser -- Order\n@enduml',
      umlImageUrl: ''
    })

    const sendMessage = async () => {
      if (!currentChat.value || userInput.value.trim() === '') return
      
      currentChat.value.messages.push({ type: 'user', content: userInput.value })
      
      // TODO: Implement AI interaction
      console.log('Sending message:', userInput.value)
      
      // Simulating AI response
      currentChat.value.messages.push({ type: 'ai', content: 'AI response to: ' + userInput.value })
      
      userInput.value = ''
    }

    const updateUMLDiagram = () => {
      if (!currentChat.value) return
      const encoded = plantumlEncoder.encode(currentChat.value.plantUMLCode)
      currentChat.value.umlImageUrl = `http://www.plantuml.com/plantuml/img/${encoded}`
    }

    const copyCode = () => {
      if (!currentChat.value) return
      navigator.clipboard.writeText(currentChat.value.plantUMLCode)
        .then(() => alert('Code copied to clipboard!'))
        .catch(err => console.error('Failed to copy code: ', err))
    }

    const exportJPEG = () => {
      const umlImage = document.querySelector('.uml-diagram img')
      if (umlImage) {
        const canvas = document.createElement('canvas')
        canvas.width = umlImage.naturalWidth
        canvas.height = umlImage.naturalHeight
        canvas.getContext('2d').drawImage(umlImage, 0, 0)
        
        canvas.toBlob((blob) => {
          const url = URL.createObjectURL(blob)
          const a = document.createElement('a')
          a.href = url
          a.download = 'uml-diagram.jpg'
          a.click()
          URL.revokeObjectURL(url)
        }, 'image/jpeg')
      }
    }

    const newChat = () => {
      chats.value.push(createNewChat())
      currentChatIndex.value = chats.value.length - 1
      updateUMLDiagram()
    }

    const switchChat = (index) => {
      currentChatIndex.value = index
    }

    onMounted(() => {
      newChat() // Create the first chat
    })

    watch(currentChat, () => {
      if (currentChat.value) {
        updateUMLDiagram()
      }
    }, { deep: true })

    return {
      userInput,
      chats,
      currentChatIndex,
      currentChat,
      sendMessage,
      updateUMLDiagram,
      copyCode,
      exportJPEG,
      newChat,
      switchChat
    }
  }
}
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  display: flex;
  flex-direction: column;
  height: 100vh;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background-color: #f0f0f0;
}

main {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.chat-nav {
  width: 200px;
  padding: 1rem;
  background-color: #f8f8f8;
  overflow-y: auto;
}

.chat-list {
  list-style-type: none;
  padding: 0;
}

.chat-list li {
  cursor: pointer;
  padding: 0.5rem;
  margin-bottom: 0.5rem;
  background-color: #e0e0e0;
  border-radius: 4px;
}

.chat-list li.active {
  background-color: #d0d0d0;
}

.chat-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  overflow: hidden;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.chat-messages .user,
.chat-messages .ai {
  max-width: 80%;
  padding: 0.5rem 1rem;
  margin-bottom: 0.5rem;
  border-radius: 1rem;
}

.chat-messages .user {
  align-self: flex-end;
  background-color: #007bff;
  color: white;
}

.chat-messages .ai {
  align-self: flex-start;
  background-color: #6c757d;
  color: white;
}

.chat-input {
  display: flex;
  margin-top: 1rem;
}

.chat-input input {
  flex: 1;
  padding: 0.5rem;
}

.editor-preview {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.code-preview,
.uml-diagram {
  flex: 1;
  overflow: auto;
}

.uml-diagram img {
  max-width: 100%;
  height: auto;
}

.btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.new-chat {
  margin-bottom: 1rem;
}
</style>