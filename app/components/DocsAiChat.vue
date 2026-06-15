<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits(['close'])

interface Message {
  role: 'user' | 'assistant'
  content: string
}

const input = ref('')
const isTyping = ref(false)

const messages = ref<Message[]>([
  {
    role: 'user',
    content: 'hai'
  },
  {
    role: 'assistant',
    content: `Hello! Docus provides a powerful framework for building modern documentation websites with Nuxt and Markdown.

Whether you are starting a new project or looking to enhance your existing docs, Docus offers a variety of tools to help you succeed:

**Get Started**

- **[Introduction](#)**: Learn about the core vision and features of Docus.
- **[Installation](#)**: Follow the steps to get your first Docus site up and running.`
  }
])

const clearChat = () => {
  messages.value = []
}

const sendMessage = () => {
  if (!input.value.trim()) return

  messages.value.push({
    role: 'user',
    content: input.value
  })

  const question = input.value
  input.value = ''
  isTyping.value = true

  // Simulate AI response
  setTimeout(() => {
    messages.value.push({
      role: 'assistant',
      content: `I'm an AI assistant simulation. You asked: "${question}".\n\nIn a real application, this would connect to an AI API endpoint (like OpenAI or Anthropic) to provide relevant documentation answers.`
    })
    isTyping.value = false
  }, 1000)
}
</script>

<template>
  <div class="flex flex-col h-full">
    <!-- Header -->
    <div class="flex items-center justify-between pb-4 mb-4 border-b border-white/5 shrink-0">
      <h3 class="text-base font-semibold text-white">Ask AI</h3>
      <div class="flex items-center gap-3 text-zinc-400">
        <button class="hover:text-white transition-colors" title="Expand">
          <UIcon name="i-lucide-maximize-2" class="w-4 h-4" />
        </button>
        <button class="hover:text-white transition-colors" title="Clear chat" @click="clearChat">
          <UIcon name="i-lucide-trash-2" class="w-4 h-4" />
        </button>
        <button class="hover:text-white transition-colors" title="Close AI" @click="emit('close')">
          <UIcon name="i-lucide-x" class="w-4 h-4" />
        </button>
      </div>
    </div>

    <!-- Chat History -->
    <div class="flex-1 overflow-y-auto mb-4 flex flex-col gap-6 no-scrollbar pr-2">
      <div v-if="messages.length === 0" class="text-center text-sm text-zinc-500 mt-10">
        Ask a question about the documentation...
      </div>

      <template v-for="(msg, index) in messages" :key="index">
        <!-- User Message -->
        <div v-if="msg.role === 'user'" class="self-end bg-white/5 text-sm text-white px-3 py-2 rounded-lg max-w-[85%] border border-white/5">
          {{ msg.content }}
        </div>

        <!-- AI Message -->
        <div v-else class="text-sm text-zinc-300">
          <div v-if="index === 1" class="text-[10px] text-zinc-500 mb-3 tracking-wide">
            Sources used<br/>
            <span class="text-zinc-400 text-xs">• Listed documentation pages</span>
          </div>
          
          <!-- We use MDC (Markdown Component) or a simple prose to render AI response -->
          <div class="prose prose-invert prose-emerald prose-sm max-w-none ai-prose">
            <MDC :value="msg.content" />
          </div>
        </div>
      </template>

      <!-- Typing Indicator -->
      <div v-if="isTyping" class="text-sm text-zinc-500 flex items-center gap-1">
        <UIcon name="i-lucide-loader-2" class="w-4 h-4 animate-spin" /> AI is thinking...
      </div>
    </div>

    <!-- Input Area -->
    <div class="mt-auto shrink-0 relative">
      <div class="border border-white/10 rounded-xl bg-black/40 p-3 focus-within:border-emerald-500/50 focus-within:bg-black/60 transition-colors">
        <textarea
          v-model="input"
          class="w-full bg-transparent text-sm text-white resize-none outline-none min-h-[60px] placeholder:text-zinc-600"
          placeholder="Ask a question..."
          @keydown.enter.prevent="sendMessage"
        />
        <div class="flex items-center justify-between mt-2">
          <div class="text-xs text-zinc-600 flex items-center gap-1">
            Line break <UKbd class="bg-white/5 border-white/10 text-zinc-400 px-1.5 py-0.5 min-w-0">⇧</UKbd> <UKbd class="bg-white/5 border-white/10 text-zinc-400 px-1.5 py-0.5 min-w-0">↵</UKbd>
          </div>
          <button 
            @click="sendMessage"
            :disabled="!input.trim() || isTyping"
            class="bg-emerald-500 hover:bg-emerald-400 disabled:opacity-50 disabled:hover:bg-emerald-500 text-black p-1.5 rounded-md transition-colors"
          >
            <UIcon name="i-lucide-arrow-up" class="w-4 h-4" />
          </button>
        </div>
      </div>
      <div class="flex items-center justify-between text-[10px] text-zinc-600 mt-2 px-1">
        <span>Chat is cleared on refresh</span>
        <span>{{ input.length }}/1000</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Custom prose styles for AI chat to match the screenshot */
:deep(.ai-prose p) {
  margin-top: 0;
  margin-bottom: 1rem;
  line-height: 1.6;
}
:deep(.ai-prose strong) {
  color: white;
}
:deep(.ai-prose a) {
  color: #10b981;
  text-decoration: none;
  font-weight: 500;
}
:deep(.ai-prose a:hover) {
  text-decoration: underline;
}
:deep(.ai-prose ul) {
  margin-top: 0.5rem;
  padding-left: 1rem;
}
:deep(.ai-prose li) {
  margin-bottom: 0.5rem;
}
:deep(.ai-prose li::marker) {
  color: #52525b; /* zinc-600 */
}
</style>
