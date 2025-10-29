<template>
    <!-- Chat History -->
    <v-container fluid class="chat-container">
        <v-row v-if="chat_history.length > 0">
            <v-col>
                <div v-for="(msg, index) in chat_history" :key="index" class="my-2">
                    <v-card elevation="2" rounded="xl"
                        :class="msg.role === 'user' ? 'pa-3 bg-gray lighten-5 ml-auto' : 'pa-3 bg-red lighten-5 mr-auto'"
                        outlined max-width="40%">
                        <v-card-subtitle class="pa-0 pb-1 d-flex align-center justify-space-between">
                            <span>{{ msg.role === 'user' ? '👤 You' : '🤖 Assistant' }}</span>

                            <!-- Tools indicator for assistant messages -->
                            <v-menu v-if="msg.role === 'assistant' && msg.tools && msg.tools.length > 0"
                                location="bottom">
                                <template v-slot:activator="{ props }">
                                    <v-chip v-bind="props" size="small" color="blue" variant="flat"
                                        prepend-icon="mdi-tools" class="ml-2 text-white elevation-2">
                                        {{ msg.tools.length }} {{ msg.tools.length === 1 ? 'tool' : 'tools' }}
                                    </v-chip>
                                </template>

                                <v-card max-width="400" class="pa-2">
                                    <v-card-title class="text-subtitle-1 pb-2">
                                        🔧 Tools Used
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
                                                    <span class="text-caption" :title="value">{{ value }}</span>
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
                            <span>🤖 Assistant</span>
                        </v-card-subtitle>
                        <div class="d-flex align-center">
                            <v-progress-circular indeterminate size="20" width="2" class="mr-2"></v-progress-circular>
                            <span>Typing...</span>
                        </div>
                    </v-card>
                </div>
            </v-col>
        </v-row>

        <v-row v-else class="fill-height">
            <v-col class="d-flex justify-center align-center flex-column">
                <div class="welcome-container text-center">
                    <v-icon size="80" color="red-darken-1" class="mb-4">mdi-robot-outline</v-icon>
                    <h1 class="welcome-text mb-2" title="Your web interface for testing AI agents">
                        {{ displayedText }}<span class="cursor">|</span>
                    </h1>
                    <p class="subtitle-text mb-6">Start a conversation with your AI assistant ✨</p>

                    <div class="feature-cards d-flex flex-wrap justify-center gap-4">
                        <v-card class="feature-card ma-2" elevation="2" rounded="lg" width="200">
                            <v-card-text class="text-center pa-4">
                                <v-icon color="blue" size="24" class="mb-2">mdi-chat-processing</v-icon>
                                <div class="text-subtitle-2 font-weight-bold">Smart Conversations</div>
                                <div class="text-caption text-medium-emphasis">Natural language interactions</div>
                            </v-card-text>
                        </v-card>

                        <v-card class="feature-card ma-2" elevation="2" rounded="lg" width="200">
                            <v-card-text class="text-center pa-4">
                                <v-icon color="green" size="24" class="mb-2">mdi-tools</v-icon>
                                <div class="text-subtitle-2 font-weight-bold">Tool Integration</div>
                                <div class="text-caption text-medium-emphasis">Enhanced capabilities</div>
                            </v-card-text>
                        </v-card>

                        <v-card class="feature-card ma-2" elevation="2" rounded="lg" width="200">
                            <v-card-text class="text-center pa-4">
                                <v-icon color="purple" size="24" class="mb-2">mdi-lightning-bolt</v-icon>
                                <div class="text-subtitle-2 font-weight-bold">Fast Responses</div>
                                <div class="text-caption text-medium-emphasis">Quick and accurate answers</div>
                            </v-card-text>
                        </v-card>
                    </div>
                </div>
            </v-col>
        </v-row>
    </v-container>

    <!-- User Input -->
    <v-footer app class="bg-gray" elevation="4">
        <v-responsive class="mx-auto mb-2" max-width="800">
            <div class="d-flex align-center">
                <v-textarea elevation="2" rounded="xl" variant="solo" v-model="message" placeholder="Ask a question"
                    prepend-inner-icon="mdi-robot" auto-grow rows="1" max-rows="6" hide-details="auto"
                    @keydown.enter.prevent="sendMessage" :disabled="isLoading" class="flex-grow-1 text-input">
                </v-textarea>
                <v-btn v-if="message.trim().length > 0" @click="sendMessage" icon="mdi-send" color="red"
                    :disabled="isLoading" class="ml-2">
                </v-btn>
            </div>

            <div class="mt-2">
                <small class="d-flex align-center justify-center text--secondary">
                    Press "Enter" to send the message or click the send icon.
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
import { ref, nextTick, onMounted, onBeforeUnmount, watch } from 'vue'
import axios from 'axios'
import localforage from 'localforage'
import { v4 as uuidv4 } from 'uuid'

const message = ref('')
const chat_history = ref([])
const isLoading = ref(false)
const errorMessage = ref('')
const displayedText = ref('')
const conversationId = ref(uuidv4()) // Generate conversation ID on component creation

// Typing effect for welcome message
const fullText = 'Welcome to Hermes-AI'
let typingIndex = 0

const typeText = () => {
    if (typingIndex < fullText.length) {
        displayedText.value += fullText.charAt(typingIndex)
        typingIndex++
        setTimeout(typeText, 100)
    }
}

// Function to save chat history to localForage
const saveChatHistory = async () => {
    try {
        // Convert reactive objects to plain objects and serialize timestamps
        const serializedHistory = JSON.parse(JSON.stringify(chat_history.value)).map(msg => ({
            ...msg,
            timestamp: new Date(msg.timestamp).toISOString()
        }))

        await localforage.setItem(conversationId.value, {
            chat_history: serializedHistory,
            timestamp: new Date().toISOString(),
            id: conversationId.value
        })
        console.log('Chat history saved with ID:', conversationId.value)
    } catch (error) {
        console.error('Error saving chat history:', error)
    }
}

// Function to load chat history if "actual_chat_history" exists
const loadChatHistory = async () => {
    try {
        const existingChat = await localforage.getItem('actual_chat_history')
        if (existingChat) {
            chat_history.value = existingChat.chat_history.map(msg => ({
                ...msg,
                timestamp: new Date(msg.timestamp)
            }))
            conversationId.value = existingChat.id || uuidv4()
            console.log('Loaded existing chat history with ID:', conversationId.value)
        }
    } catch (error) {
        console.error('Error loading chat history:', error)
    }
}

// Watch for changes in chat_history and save automatically
watch(chat_history, async () => {
    if (chat_history.value.length > 0) {
        await saveChatHistory()
    }
}, { deep: true })

// Function to handle page unload
const handleBeforeUnload = async () => {
    if (chat_history.value.length > 0) {
        await saveChatHistory()
    }
}

// Add listener when component is mounted
onMounted(async () => {
    window.addEventListener('beforeunload', handleBeforeUnload)
    await loadChatHistory()
    typeText()
})

// Remove listener and save chat when component is unmounted
onBeforeUnmount(async () => {
    window.removeEventListener('beforeunload', handleBeforeUnload)
    await saveChatHistory()
})

// Configure axios instance
const api = axios.create({
    baseURL: 'http://localhost:8000',
    headers: {
        'Content-Type': 'application/json',
    },
})

const sendMessage = async () => {
    if (message.value.trim() && !isLoading.value) {
        const userMessage = message.value.trim()

        // Add user message to history
        chat_history.value.push({
            role: 'user',
            content: userMessage,
            timestamp: new Date()
        })

        // Clear input
        message.value = ''

        // Scroll to bottom
        await nextTick()
        scrollToBottom()

        // Activate loading
        isLoading.value = true
        errorMessage.value = ''

        try {
            // Prepare history in the format expected by backend
            const historyForBackend = chat_history.value.map(msg => ({
                role: msg.role,
                content: msg.content
            }))

            // Make request to backend using axios
            const { data } = await api.post('/chat', {
                message: userMessage,
                chat_history: historyForBackend
            })

            // Extract assistant response
            let assistantMessage = ''
            if (data.response && data.response.blocks) {
                assistantMessage = data.response.blocks
                    .filter(block => block.block_type === 'text')
                    .map(block => block.text)
                    .join('\n')
            } else {
                assistantMessage = 'Sorry, I could not process the response.'
            }

            // Extract used tools
            const tools = data.tool_calls || []

            // Add assistant response to history with tool information
            chat_history.value.push({
                role: 'assistant',
                content: assistantMessage,
                timestamp: new Date(),
                tools: tools,
                agent_name: data.current_agent_name || 'Assistant'
            })

            // Scroll to bottom
            await nextTick()
            scrollToBottom()

        } catch (error) {
            console.error('Error sending message:', error)

            if (error.response) {
                errorMessage.value = `Server error: ${error.response.status} - ${error.response.data.error || error.response.statusText}`
            } else if (error.request) {
                errorMessage.value = 'Error: Could not connect to server. Check if the backend is running.'
            } else {
                errorMessage.value = `Error: ${error.message}`
            }

            // Remove user message if there's an error
            chat_history.value.pop()
        } finally {
            isLoading.value = false
        }
    }
}

const formatTimestamp = (timestamp) => {
    // Handle both Date objects and ISO strings
    const date = timestamp instanceof Date ? timestamp : new Date(timestamp)
    return date.toLocaleTimeString('en-US', {
        hour: '2-digit',
        minute: '2-digit'
    })
}

const formatToolName = (toolName) => {
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
    box-shadow: #ff4759 2px 1px 8px !important;
}

.text-input {
    border-radius: 24px;
    border: .5px solid #ff4759;
}

.message-content {
    white-space: pre-wrap;
    word-wrap: break-word;
}

/* Welcome text with red gradient */
.welcome-text {
    font-size: 3.5rem;
    font-weight: 900;
    background: linear-gradient(135deg, #8B0000 0%, #DC143C 20%, #FF0000 40%, #ff4759 60%, #ff6b7a 80%, #FFB6C1 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    letter-spacing: -1px;
    animation: gradient-shift 3s ease infinite;
    background-size: 200% 200%;
}

/* Gradient animation */
@keyframes gradient-shift {
    0% {
        background-position: 0% 50%;
    }

    50% {
        background-position: 100% 50%;
    }

    100% {
        background-position: 0% 50%;
    }
}

/* Blinking cursor */
.cursor {
    color: #ff4759;
    animation: blink 1s step-end infinite;
    font-weight: 400;
}

@keyframes blink {

    0%,
    100% {
        opacity: 1;
    }

    50% {
        opacity: 0;
    }
}

/* Subtitle text */
.subtitle-text {
    font-size: 1.2rem;
    color: #666;
    font-weight: 300;
}
</style>