<template>
    <!-- Chat History -->
    <v-container fluid class="chat-container">
        <v-row>
            <v-col>
                <div v-for="(msg, index) in chat_history" :key="index" class="my-2">
                    <v-card elevation="2" rounded="xl"
                        :class="msg.role === 'user' ? 'pa-3 bg-gray lighten-5 ml-auto' : 'pa-3 bg-red lighten-5 mr-auto'"
                        outlined max-width="40%">
                        <v-card-subtitle class="pa-0 pb-1 d-flex align-center justify-space-between">
                            <span>{{ msg.role === 'user' ? '👤 Você' : '🤖 Assistente' }}</span>

                            <!-- Tools indicator for assistant messages -->
                            <v-menu v-if="msg.role === 'assistant' && msg.tools && msg.tools.length > 0"
                                location="bottom">
                                <template v-slot:activator="{ props }">
                                    <v-chip v-bind="props" size="small" color="blue" variant="flat"
                                        prepend-icon="mdi-tools" class="ml-2 text-white elevation-2">
                                        {{ msg.tools.length }} {{ msg.tools.length === 1 ? 'ferramenta' : 'ferramentas'
                                        }}
                                    </v-chip>
                                </template>

                                <v-card max-width="400" class="pa-2">
                                    <v-card-title class="text-subtitle-1 pb-2">
                                        🔧 Ferramentas Utilizadas
                                    </v-card-title>
                                    <v-divider></v-divider>
                                    <v-list density="compact">
                                        <v-list-item v-for="(tool, toolIndex) in msg.tools" :key="toolIndex"
                                            class="px-2">
                                            <template v-slot:prepend>
                                                <v-icon color="blue">mdi-wrench</v-icon>
                                            </template>
                                            <v-list-item-title class="text-wrap">
                                                <strong>{{ formatToolName(tool.tool_name) }}</strong>
                                            </v-list-item-title>
                                            <v-list-item-subtitle class="text-wrap mt-1">
                                                <div v-for="(value, key) in tool.tool_kwargs" :key="key" class="my-1">
                                                    <span class="font-weight-medium">{{ key }}: </span>
                                                    <span class="text-caption">{{ value }}</span>
                                                </div>
                                            </v-list-item-subtitle>
                                        </v-list-item>
                                    </v-list>
                                </v-card>
                            </v-menu>
                        </v-card-subtitle>

                        <div class="message-content">{{ msg.content }}</div>

                        <div class="d-flex justify-end mt-1">
                            <small class="text--secondary">{{ formatTimestamp(msg.timestamp) }}</small>
                        </div>
                    </v-card>
                </div>

                <!-- Loading indicator -->
                <div v-if="isLoading" class="my-2">
                    <v-card elevation="2" rounded="xl" class="pa-3 bg-red lighten-5 mr-auto" outlined max-width="40%">
                        <v-card-subtitle class="pa-0 pb-1">
                            <span>🤖 Assistente</span>
                        </v-card-subtitle>
                        <div class="d-flex align-center">
                            <v-progress-circular indeterminate size="20" width="2" class="mr-2"></v-progress-circular>
                            <span>Digitando...</span>
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
                <v-textarea elevation="2" rounded="xl" variant="solo" v-model="message" placeholder="Faça uma pergunta"
                    prepend-inner-icon="mdi-robot" auto-grow rows="1" max-rows="6" hide-details="auto"
                    @keydown.enter.prevent="sendMessage" :disabled="isLoading" class="flex-grow-1 text-input">
                </v-textarea>
                <v-btn v-if="message.trim().length > 0" @click="sendMessage" icon="mdi-send" color="red"
                    :disabled="isLoading" class="ml-2">
                </v-btn>
            </div>

            <div class="mt-2">
                <small class="d-flex align-center justify-center text--secondary">
                    Pressione "Enter" para enviar a mensagem ou clique no ícone de envio.
                </small>
            </div>

            <!-- Error message -->
            <v-alert v-if="errorMessage" type="error" class="mt-2" closable @click:close="errorMessage = ''">
                {{ errorMessage }}
            </v-alert>
        </v-responsive>
    </v-footer>
</template>

<script setup>
import { ref, nextTick, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'

const message = ref('')
const chat_history = ref([])
const isLoading = ref(false)
const errorMessage = ref('')

// Função para alertar antes de sair da página
const handleBeforeUnload = (event) => {
    if (chat_history.value.length > 0) {
        event.preventDefault()
        event.returnValue = 'Você tem uma conversa em andamento. Se atualizar a página, todo o histórico será perdido. Deseja realmente sair?'
        return event.returnValue
    }
}

// Adiciona o listener quando o componente é montado
onMounted(() => {
    window.addEventListener('beforeunload', handleBeforeUnload)
})

// Remove o listener quando o componente é desmontado
onBeforeUnmount(() => {
    window.removeEventListener('beforeunload', handleBeforeUnload)
})

// Configura a instância do axios
const api = axios.create({
    baseURL: 'http://localhost:8000',
    headers: {
        'Content-Type': 'application/json',
    },
    timeout: 30000 // 30 segundos de timeout
})

const sendMessage = async () => {
    if (message.value.trim() && !isLoading.value) {
        const userMessage = message.value.trim()

        // Adiciona mensagem do usuário ao histórico
        chat_history.value.push({
            role: 'user',
            content: userMessage,
            timestamp: new Date()
        })

        // Limpa o input
        message.value = ''

        // Scroll para o final
        await nextTick()
        scrollToBottom()

        // Ativa loading
        isLoading.value = true
        errorMessage.value = ''

        try {
            // Prepara o histórico no formato esperado pelo backend
            const historyForBackend = chat_history.value.map(msg => ({
                role: msg.role,
                content: msg.content
            }))

            // Faz a requisição para o backend usando axios
            const { data } = await api.post('/chat', {
                message: userMessage,
                chat_history: historyForBackend
            })

            // Extrai a resposta do assistente
            let assistantMessage = ''
            if (data.response && data.response.blocks) {
                assistantMessage = data.response.blocks
                    .filter(block => block.block_type === 'text')
                    .map(block => block.text)
                    .join('\n')
            } else {
                assistantMessage = 'Desculpe, não consegui processar a resposta.'
            }

            // Extrai as ferramentas utilizadas
            const tools = data.tool_calls || []

            // Adiciona resposta do assistente ao histórico com informações das ferramentas
            chat_history.value.push({
                role: 'assistant',
                content: assistantMessage,
                timestamp: new Date(),
                tools: tools,
                agent_name: data.current_agent_name || 'Assistente'
            })

            // Scroll para o final
            await nextTick()
            scrollToBottom()

        } catch (error) {
            console.error('Erro ao enviar mensagem:', error)

            // Tratamento de erros mais detalhado
            if (error.response) {
                // O servidor respondeu com um status de erro
                errorMessage.value = `Erro do servidor: ${error.response.status} - ${error.response.data.error || error.response.statusText}`
            } else if (error.request) {
                // A requisição foi feita mas não houve resposta
                errorMessage.value = 'Erro: Não foi possível conectar ao servidor. Verifique se o backend está rodando.'
            } else {
                // Algo aconteceu na configuração da requisição
                errorMessage.value = `Erro: ${error.message}`
            }

            // Remove a mensagem do usuário se houver erro
            chat_history.value.pop()
        } finally {
            isLoading.value = false
        }
    }
}

const formatTimestamp = (timestamp) => {
    return timestamp.toLocaleTimeString('pt-BR', {
        hour: '2-digit',
        minute: '2-digit'
    })
}

const formatToolName = (toolName) => {
    // Converte snake_case para Title Case
    return toolName
        .split('_')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(' ')
}

const scrollToBottom = () => {
    const container = document.querySelector('.chat-container')
    if (container) {
        container.scrollTop = container.scrollHeight
    }
}
</script>

<style scoped>
.chat-container {
    height: calc(100vh - 150px);
    overflow-y: auto;
    padding-bottom: 20px;
}

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

.message-content {
    white-space: pre-wrap;
    word-wrap: break-word;
}
</style>