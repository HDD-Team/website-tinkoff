<template>
  <div class="relative flex h-screen bg-tinkoffYellow">
    <!-- Левая реклама -->
    <a
      href="https://www.tinkoff.ru/loans/pos-loans/"
      target="_blank"
      class="flex-shrink-0 w-1/6 p-4"
    >
      <img
        src="@/assets/Tinkoff banner Photo 9_16.png"
        alt="Реклама Тинькофф 1"
        class="w-full h-auto object-cover rounded-lg shadow-md"
      />
    </a>

    <!-- Основной контейнер чата -->
    <div
      class="flex flex-col flex-1 max-w-4xl mx-auto border border-gray-300 rounded-lg shadow-lg bg-white"
    >
      <!-- Логотип и заголовок Тинькофф -->
      <div class="p-4 flex items-center justify-between">
        <h1 class="text-2xl font-bold text-black">Тинькофф Помощь</h1>
        <a href="https://www.tinkoff.ru/help/" target="_blank">
          <img
            src="@/assets/tinkoff_logo.png"
            alt="Логотип Тинькофф"
            class="h-12 ml-4 hover:opacity-75"
          />
        </a>
      </div>

      <div class="flex-1 p-4 overflow-y-auto bg-gray-50">
        <div v-for="(message, index) in messages" :key="message.id" class="mb-4 flex items-start">
          <div v-if="message.user" class="flex items-center ml-auto">
            <span class="inline-block bg-gray-300 text-gray-800 py-2 px-4 rounded-lg shadow-sm">
              {{ message.text }}
            </span>
            <img
              src="@/assets/path_to_user_icon.png"
              alt="Пользователь"
              class="w-8 h-8 rounded-full ml-2 shadow-sm"
            />
          </div>
          <div v-else class="flex items-start">
            <img
              v-if="!message.typing"
              src="@/assets/path_to_bot_icon.png"
              alt="Бот"
              class="w-8 h-8 rounded-full mr-2 shadow-sm"
            />
            <div>
              <span
                v-if="!message.typing"
                class="inline-block bg-yellow-400 text-black py-2 px-4 rounded-lg shadow-sm"
              >
                {{ message.text }}
              </span>
              <span
                v-else
                class="inline-block bg-yellow-400 text-black py-2 px-4 rounded-lg shadow-sm"
              >
                Бот печатает...
              </span>
              <div v-if="message.links && message.links.length" class="mt-2">
                <div v-for="(link, linkIndex) in message.links" :key="linkIndex" class="mt-1">
                  <a
                    :href="link"
                    target="_blank"
                    class="inline-flex items-center text-sm text-blue-600 hover:underline"
                  >
                    {{ link }}
                  </a>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="p-4 border-t border-gray-300 bg-white flex items-center">
        <input
          v-model="input"
          @keyup.enter="sendMessage"
          type="text"
          placeholder="Написать сообщение..."
          class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-yellow-400"
        />
        <button
          @click="sendMessage"
          class="ml-2 p-2 bg-yellow-400 text-black rounded-full flex items-center justify-center shadow-lg hover:bg-yellow-500 focus:outline-none focus:ring-2 focus:ring-yellow-400"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="w-6 h-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M5 12h14M12 5l7 7-7 7"
            />
          </svg>
        </button>
      </div>
    </div>

    <!-- Правая реклама -->
    <a
      href="https://www.tinkoff.ru/loans/pos-loans/"
      target="_blank"
      class="flex-shrink-0 w-1/6 p-4"
    >
      <img
        src="@/assets/Tinkoff banner 3d 9_16.png"
        alt="Реклама Тинькофф 2"
        class="w-full h-auto object-cover rounded-lg shadow-md"
      />
    </a>
  </div>
</template>

<script>
import '@/assets/tinkoff-fonts.css'
import axios from 'axios'
import { v4 as uuidv4 } from 'uuid'

export default {
  data() {
    return {
      messages: [
        {
          id: uuidv4(),
          text: 'Привет, это QnA бот Тинькофф Помощь – Бизнес. Задавай мне вопросы связанные с бизнесом.',
          user: false,
          links: []
        }
      ],
      input: ''
    }
  },
  methods: {
    async sendMessage() {
      if (this.input.trim() === '') return
      const userMessage = { id: uuidv4(), text: this.input, user: true }
      this.messages.push(userMessage)
      this.input = ''

      const botTyping = {
        id: uuidv4(),
        text: '',
        user: false,
        typing: true,
        relatedTo: userMessage.id
      }
      this.messages.push(botTyping)

      try {
        const response = await axios.post('http://46.147.127.169:8000/assist', {
          query: userMessage.text
        })
        this.messages = this.messages.map((msg) => {
          if (msg.id === botTyping.id) {
            return { ...msg, text: response.data.answer, typing: false, links: [response.data.url] }
          }
          return msg
        })
      } catch (error) {
        this.messages = this.messages.map((msg) => {
          if (msg.id === botTyping.id) {
            return { ...msg, text: 'Ошибка связи с сервером.', typing: false }
          }
          return msg
        })
      }
    }
  }
}
</script>
