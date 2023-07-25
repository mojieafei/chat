<template>
  <div>
    <h1>Chat Service</h1>
    <div v-if="!connected">
      <h2>Connect to Server</h2>
      <form @submit.prevent="connect">
        <label for="host">Host:</label>
        <input type="text" id="host" v-model="host" required>
        <label for="port">Port:</label>
        <input type="number" id="port" v-model="port" required>
        <button type="submit">Connect</button>
      </form>
    </div>
    <div v-else>
      <h2>Connected to Server</h2>
      <form @submit.prevent="sendMessage">
        <label for="message">Message:</label>
        <input type="text" id="message" v-model="message" required>
        <button type="submit">Send</button>
      </form>
      <ul>
        <li v-for="(message, index) in messages" :key="index">{{ message }}</li>
      </ul>
    </div>
  </div>
</template>

<script>
import { reactive } from 'vue'
import { grpc } from '@improbable-eng/grpc-web'
import { ChatService } from './chat_pb_service'
import { ChatRequest, ChatResponse } from './chat_pb'

const chatService = new ChatService('http://localhost:8080', grpc.credentials.createInsecure())

export default {
  setup() {
    const state = reactive({
      connected: false,
      host: '',
      port: '',
      message: '',
      messages: []
    })

    const connect = () => {
      chatService.connect({ host: state.host, port: state.port }, (err, response) => {
        if (err) {
          console.error(err)
          return
        }
        console.log(response)
        state.connected = true
      })
    }

    const sendMessage = () => {
      const request = new ChatRequest()
      request.setMessage(state.message)
      chatService.sendMessage(request, (err, response) => {
        if (err) {
          console.error(err)
          return
        }
        console.log(response)
        state.messages.push(response.getMessage())
        state.message = ''
      })
    }

    return { state, connect, sendMessage }
  }
}
</script>
