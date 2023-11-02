<template>
  <div class="chat-window">
    <div class="messages">
      <div v-for="message in messages" :key="message.id" class="message">
        <div v-if="message.sender === 'user'" class="user-message">
          {{ message.text }}
        </div>
        <div v-else class="bot-message">
          {{ message.text }}
        </div>
      </div>
    </div>
    <div class="input-area">
      <input v-model="userInput" @keyup.enter="sendMessage" placeholder="输入您的消息"/>
      <button @click="sendMessage">发送</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      userInput: '',
      messages: [],
    };
  },
  methods: {
    async sendMessage() {
      const userMessage = {
        id: Date.now(),
        text: this.userInput,
        sender: 'user',
      };

      this.messages.push(userMessage);
      this.userInput = '';

      const botMessage = await this.chatResp(userMessage.text);
      this.messages.push(
          {
            id: Date.now(),
            text: botMessage,
            sender: 'bot'
          }
      );
    },
    async chatResp(input) {
      return (await axios.post('api/chat/fastchat',
          JSON.stringify({
            "model": "baichuan2-13b-int4",
            "messages": [
              {
                "role": "user",
                "content": input
              }
            ],
            "temperature": 0.7,
            "n": 1,
            "max_tokens": 1024,
            "stop": [],
            "stream": false,
            "presence_penalty": 0,
            "frequency_penalty": 0
          }),
          {
            headers: {
              'Content-Type': 'application/json',
            },
          }
      )).data
    }
    //
    // async getBotResponse(userInput) {
    //   const response = await axios.post('http://10.70.25.127:15009/v1/chat/completions', {
    //     message: userInput,
    //     max_tokens: 50,
    //   }, {
    //     headers: {
    //       Authorization: 'xxx', // 替换为您的API密钥
    //     },
    //   });
    //
    //   return {
    //     id: Date.now(),
    //     text: response.data.choices[0].text.trim(),
    //     sender: 'bot',
    //   };
    // },
  },
};
</script>

<style scoped>
.chat-window {
  width: 400px;
  height: 500px;
  border: 1px solid #ccc;
  padding: 10px;
  display: flex;
  flex-direction: column;
}

.messages {
  flex-grow: 1;
  overflow-y: auto;
}

.message {
  margin-bottom: 10px;
}

.user-message {
  background-color: #f0f0f0;
  padding: 5px;
}

.bot-message {
  background-color: #e0e0e0;
  padding: 5px;
}

.input-area {
  display: flex;
  margin-top: 10px;
}

.input-area input {
  flex-grow: 1;
  margin-right: 10px;
}

.input-area button {
  padding: 5px 10px;
}
</style>
