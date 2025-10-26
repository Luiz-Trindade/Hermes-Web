<template>
    <!-- Chat History -->
    <v-container fluid>
        <v-row>
            <v-col>
                <div v-for="(msg, index) in chat_history" :key="index" class="my-2">
                    <v-card elevation="2" rounded="xl"
                        :class="msg.role === 'user' ? 'pa-3 bg-gray lighten-5 ml-auto' : 'pa-3 bg-red lighten-5 mr-auto'"
                        outlined max-width="40%">
                        <v-card-subtitle class="pa-0 pb-1">
                            <span>{{ msg.role === 'user' ? '👤 Você' : '🤖 Assistente' }}</span>
                        </v-card-subtitle>
                        <div>{{ msg.content }}</div>
                        <div class="d-flex justify-end mt-1">
                            <small class="text--secondary">{{ formatTimestamp(msg.timestamp) }}</small>
                        </div>
                    </v-card>
                </div>
            </v-col>
        </v-row>
    </v-container>

    <!-- User Input -->
    <v-footer app class="bg-gray" elevation="4">
        <v-responsive class="mx-auto mb-2" max-width="800">
            <div class="d-flex align-center">
                <v-textarea elevation="2" rounded="xl" variant="solo" v-model="message" placeholder="Ask some question"
                    prepend-inner-icon="mdi-robot" auto-grow rows="1" max-rows="6" hide-details="auto"
                    @keydown.enter.prevent="sendMessage" class="flex-grow-1 text-input"></v-textarea>
                <v-btn v-if="message.trim().length > 0" @click="sendMessage" icon="mdi-send" color="red"
                    class="ml-2"></v-btn>
            </div>

            <div class="mt-2">
                <small class="d-flex align-center justify-center text--secondary">
                    Pressione "Enter" para enviar a mensagem ou clique no ícone de envio.
                </small>
            </div>
        </v-responsive>
    </v-footer>
</template>

<script setup>
import { ref } from 'vue'

const message = ref('')
const chat_history = ref([
    { "role": "user", "content": "Hello!", "timestamp": new Date() },
    { "role": "assistant", "content": "Hi there! How can I assist you today?", "timestamp": new Date() }
])

const sendMessage = () => {
    if (message.value.trim()) {
        chat_history.value.push({
            role: 'user',
            content: message.value.trim(),
            timestamp: new Date()
        })
        message.value = ''
    }
}

const formatTimestamp = (timestamp) => {
    return timestamp.toLocaleTimeString('pt-BR', {
        hour: '2-digit',
        minute: '2-digit'
    })
}
</script>

<style scoped>
/* Mantém o footer fixo e ajusta largura */
.v-footer {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    padding: 8px 16px;
}

.text-input {
    border-radius: 24px;
    border: .5px solid #ff4759;
}
</style>
