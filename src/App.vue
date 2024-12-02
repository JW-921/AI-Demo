<script setup>
import { ref, computed, nextTick } from 'vue';
import { callOpenAI } from './ai';
// 使用 ref 定義響應式變數
const newMessage = ref('');
const messages = ref([]);
const isTyping = ref(false);
const chatContainer = ref(null);
const statusClass = computed(() => {
  return {
    'w-3 h-3 rounded-full': true,
    'bg-green-400': !isTyping.value,
    'bg-yellow-400': isTyping.value
  }
});
const formatTime = (timestamp) => {
  return new Intl.DateTimeFormat('zh-TW', {
    hour: '2-digit',
    minute: '2-digit'
  }).format(timestamp);  
}
const scrollToBottom = async () => {
  await nextTick();
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
  }
}
const simulateAIResponse = async () => {
  isTyping.value = true
  // 模擬 AI 思考時間
  await new Promise(resolve => setTimeout(resolve, 1500));  
  const aiResponses = [
    '我了解你的問題，讓我想想...',
    '這是個很好的問題！',
    '根據我的分析...',
    '我可以協助你解決這個問題'
  ]
  const randomResponse = aiResponses[Math.floor(Math.random() * aiResponses.length)];
  messages.value.push({
    id: Date.now(),
    content: randomResponse,
    isUser: false,
    timestamp: new Date()
  });
  isTyping.value = false;
  scrollToBottom();
}
const sendMessage = async () => {
  // 新增使用者訊息
  messages.value.push({
    id: Date.now(),
    content: newMessage.value,
    isUser: true,
    timestamp: new Date()
  });
  const userMsg = newMessage.value;
  newMessage.value = '';
  // 觸發 AI 回應  
  // await simulateAIResponse();
  isTyping.value = true;
  const aiResponse = await callOpenAI(userMsg);
  messages.value.push({
    id: Date.now(),
    content: aiResponse,
    isUser: false,
    timestamp: new Date()
  });
  isTyping.value = false;
}
</script>

<template>
  <div class="container mx-auto max-w-4xl h-screen p-4">
    <div class="bg-white rounded-lg shadow-lg h-full flex flex-col">
      <!-- 標題列 -->
      <div class="bg-indigo-600 text-white px-6 py-4 rounded-t-lg flex items-center justify-between">
        <div class="flex items-center space-x-2">
          <div :class="statusClass"></div>
          <h1 class="text-xl font-bold">Chat AI 助理</h1>
        </div>
        <div class="text-sm">總共 {{ messages.length }} 則訊息</div>
      </div>

      <!-- 對話區域 -->
      <div ref="chatContainer" class="flex-1 overflow-y-auto p-4 space-y-4">

        <template v-for="msg in messages">
          <!-- AI 的訊息 -->
          <div v-if="!msg.isUser" class="flex items-start space-x-3">
            <div class="w-8 h-8 rounded-full bg-indigo-500 flex items-center justify-center text-white">
                AI
            </div>
            <div class="bg-gray-100 rounded-lg p-3 max-w-[80%]">
                <p class="text-gray-800">{{ msg.content }}</p>
                <span class="text-xs text-gray-500">{{ formatTime(msg.timestamp) }}</span>
            </div>
          </div>

          <!-- 使用者的訊息 -->
          <div v-else class="flex items-start space-x-3 justify-end">
              <div class="bg-indigo-600 text-white rounded-lg p-3 max-w-[80%]">
                <p>{{ msg.content }}</p>
                <span class="text-xs text-gray-300">{{ formatTime(msg.timestamp) }}</span>
              </div>
              <div class="w-8 h-8 rounded-full bg-gray-400 flex items-center justify-center text-white">
                你
              </div>
          </div>
        </template>
      </div>

      <!-- 輸入區域 -->
      <div class="border-t p-4">
        <div class="flex space-x-4">
          <input 
            v-model="newMessage"
            :disabled="isTyping"
            type="text"
            class="flex-1 border rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
            :placeholder="isTyping ? '電腦思考中...' : '輸入訊息...'"
            @keydown.enter="sendMessage()"
            >
          <button 
            @click="sendMessage()"
            :disabled="isTyping"
            class="bg-indigo-600 text-white px-6 py-2 rounded-lg hover:bg-indigo-700 transition-colors duration-200 disabled:opacity-50">
            送出
          </button>
        </div>
      </div>
    </div>
  </div>
</template>