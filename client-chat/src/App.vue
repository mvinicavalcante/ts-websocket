<script setup>
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
import { io } from 'socket.io-client'
import { onBeforeMount, ref } from 'vue'

const socket = io('http://localhost:3000')

const messages = ref([])
const messageText = ref('')
const joined = ref(false)
const name = ref('')
const typingDisplay = ref('')

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response
  })

  socket.on('message', (message) => {
    messages.value.push(message)
  })

  socket.on('typing', ({ name, isTyping }) => {
    typingDisplay.value = isTyping ? `${name} is typing...` : ''
  })
})

const joinChat = () => {
  socket.emit('join', { name: name.value }, () => {
    joined.value = true
  })
}

const sendMessage = () => {
  socket.emit('createMessage', { name: name.value, text: messageText.value }, () => {
    messageText.value = ''
  })
}

let timeout
const typing = () => {
  socket.emit('typing', { isTyping: true })

  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false })
  }, 2000)

}

</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="joinChat">
        <label>What's your name?</label>
        <input v-model="name" type="text" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay">{{  typingDisplay }}</div>

      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="typing" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat {
  padding: 20px;
  height: 100vh;
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages-container {
  flex: 1;
}
</style>
