<template>
  <div class="app-container">
    <!-- 侧边栏 -->
    <div class="sidebar">
      <!-- 专家身份选择 -->
      <div>
        <h2>CIM Chat Robot</h2>
      </div>
      <div class="select-container">
        <h5>选择模型：</h5>
        <select v-model="selectedOption" class="custom-select">
          <option disabled value="">请选择一个模型</option>
          <option v-for="option in options" :value="option.value" :key="option.value">
            {{ option.text }}
          </option>
        </select>
      </div>
      <div class="expert-buttons">
        <h5>选择身份：</h5>
        <button 
          v-for="expert in experts" 
          :key="expert.id"
          :class="['expert-btn', { active: currentExpert === expert.id }]"
          @click="selectExpert(expert.id)"
        >
          {{ expert.name }}
        </button>
      </div>
      
      <!-- 历史对话记录 -->
      <div class="chat-history">
        <div class="history-header">
          <h3>历史记录</h3>
          <button @click="clearHistory" class="clear-btn">
            <i class="fas fa-trash"></i>
          </button>
        </div>
        <div class="history-list">
          <div 
            v-for="(chat, index) in chatHistory" 
            :key="index"
            class="history-item"
          >
            <span class="history-time">{{ chat.time }}</span>
            <div class="history-content">{{ chat.content }}</div>
          </div>
        </div>
      </div>
    </div>

    <!-- 主聊天区域 -->
    <div class="main-chat">
      <div class="chat-header">
        
        <div class="current-expert">
          <span class="expert-label">当前模式:</span>
          <span class="expert-value">{{ getCurrentExpertName }}</span>
        </div>
      </div>
      
      <div class="chat-messages" ref="messageContainer">
        <div 
          v-for="(message, index) in currentChat" 
          :key="index"
          :class="['message', message.role]"
        >
          <div class="message-avatar">
            {{ message.role === 'user' ? '👤' : '🤖' }}
          </div>
          <div class="message-content">{{ message.content }}</div>
        </div>
      </div>

      <div class="chat-input">
        <textarea 
          v-model="userInput"
          @keyup.enter.prevent="sendMessage"
          placeholder="输入您的问题..."
          :disabled="isLoading"
        ></textarea>
        <button @click="sendMessage" :disabled="isLoading">
          <i class="fas fa-paper-plane"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
export default {
  name: 'App',
  data() {
    return {
      experts: [
        { id: 'code', name: 'LIT专家' },
        { id: 'math', name: 'MFG专家' },
        { id: 'general', name: '通识专家' },
        { id: 'history', name: 'PIE专家' }
      ],
      currentExpert: 'code',
      chatHistory: [],
      currentChat: [],
      userInput: '',
      selectedOption :'Deepseek-R1-Distill-Qwen-7B',
      options :[
      { text: 'Deepseek-R1-Distill-Qwen-7B', value: 'Deepseek-R1-Distill-Qwen-7B' },
      { text: 'Deepseek-R1-Distill-Llama-8B', value: 'Deepseek-R1-Distill-Llama-8B' },
      { text: 'Deepseek-R1-Distill-Qwen-14B', value: 'Deepseek-R1-Distill-Qwen-14B' }
    ],
      isLoading: false
    }
  },
  computed: {
    getCurrentExpertName() {
      const expert = this.experts.find(e => e.id === this.currentExpert)
      return expert ? expert.name : ''
    }
  },
  methods: {
    async sendMessage() {
      if (!this.userInput.trim() || this.isLoading) return

      const userMessage = this.userInput.trim()
      this.isLoading = true

      // 添加用户消息
      this.currentChat.push({
        role: 'user',
        content: userMessage
      })

      // 添加到历史记录
      this.chatHistory.unshift({
        time: new Date().toLocaleTimeString(),
        content: userMessage
      })

      // 清空输入
      this.userInput = ''

      try {
        // 模拟API调用
        await new Promise(resolve => setTimeout(resolve, 1000))
        
        // 这里替换为实际的API调用
        // const response = await fetch('http://your-backend-url/chat', {
        //   method: 'POST',
        //   headers: {
        //     'Content-Type': 'application/json'
        //   },
        //   body: JSON.stringify({
        //     message: userMessage,
        //     expert: this.currentExpert
        //   })
        // })
        // const data = await response.json()

        // 添加AI回复
        this.currentChat.push({
          role: 'assistant',
          content: '服务器正忙，请稍等。'
        })
      } catch (error) {
        console.error('Error:', error)
        this.currentChat.push({
          role: 'assistant',
          content: '服务器连接失败，请稍后重试。'
        })
      } finally {
        this.isLoading = false
        this.$nextTick(() => {
          this.$refs.messageContainer.scrollTop = this.$refs.messageContainer.scrollHeight
        })
      }
    },
    selectExpert(expertId) {
      this.currentExpert = expertId
    },
    clearHistory() {
      this.chatHistory = []
      this.currentChat = []
    }
  }
}
</script>

<style scoped>
.app-container {
  display: flex;
  height: 100vh;
  background-color: #1a1a1a;
  color: #ffffff;
  font-family: 'Arial', sans-serif;
}

.sidebar {
  width: 280px;
  background-color: #2a2a2a;
  display: flex;
  flex-direction: column;
  border-right: 1px solid #3a3a3a;
}

.expert-buttons {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.expert-btn {
  padding: 12px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background-color: #3a3a3a;
  color: #ffffff;
  transition: all 0.3s ease;
  font-size: 14px;
}

.expert-btn:hover {
  background-color: #4a4a4a;
}

.expert-btn.active {
  background-color: #007bff;
  box-shadow: 0 0 10px rgba(0, 123, 255, 0.3);
}

.chat-history {
  flex: 1;
  padding: 20px;
  border-top: 1px solid #3a3a3a;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.history-item {
  padding: 12px;
  margin-bottom: 8px;
  background-color: #3a3a3a;
  border-radius: 8px;
  font-size: 14px;
}

.history-time {
  font-size: 12px;
  color: #888;
  display: block;
  margin-bottom: 5px;
}

.main-chat {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: #1a1a1a;
}

.chat-header {
  padding: 20px;
  border-bottom: 1px solid #3a3a3a;
  background-color: #2a2a2a;
}

.chat-header h1 {
  margin: 0;
  font-size: 24px;
  color: #007bff;
}

.current-expert {
  margin-top: 10px;
  font-size: 14px;
}

.expert-label {
  color: #888;
}

.expert-value {
  color: #007bff;
  margin-left: 8px;
}

.chat-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
}

.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 20px;
  gap: 12px;
}

.message-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #3a3a3a;
  font-size: 20px;
}

.message-content {
  flex: 1;
  padding: 12px;
  border-radius: 12px;
  max-width: 80%;
  line-height: 1.5;
}

.message.user .message-content {
  background-color: #007bff;
  margin-left: auto;
}

.message.assistant .message-content {
  background-color: #3a3a3a;
}

.chat-input {
  padding: 20px;
  border-top: 1px solid #3a3a3a;
  display: flex;
  gap: 12px;
  background-color: #2a2a2a;
}

.chat-input textarea {
  flex: 1;
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #3a3a3a;
  resize: none;
  height: 60px;
  background-color: #3a3a3a;
  color: #ffffff;
  font-size: 14px;
}

.chat-input textarea:focus {
  outline: none;
  border-color: #007bff;
}

.chat-input button {
  padding: 0 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.chat-input button:hover {
  background-color: #0056b3;
}

.chat-input button:disabled {
  background-color: #3a3a3a;
  cursor: not-allowed;
}

.clear-btn {
  padding: 8px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.clear-btn:hover {
  background-color: #c82333;
}

/* 滚动条样式 */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #2a2a2a;
}

::-webkit-scrollbar-thumb {
  background: #3a3a3a;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #4a4a4a;
}
.custom-select {
  display: flex;
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #fff;
  align-items: center;
  
}
.select-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background: #ffffff;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}
</style> 