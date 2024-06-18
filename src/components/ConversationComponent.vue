<template>
  <div id="conversation">
    <div class="conversation-container">
      <div v-for="message in messages" :key="message.sid" class="bubble-container"
        :class="{ myMessage: message.author === name }">
        <div class="bubble">
          <div class="name">{{ message.author }}:</div>
          <div class="message">{{ message.body }}</div>
        </div>
      </div>
    </div>
    <div class="input-container">
      <input v-model="messageText" @keyup.enter="sendMessage" placeholder="Enter your message" />
      <button @click="sendMessage" :disabled="isSending">{{ isSending ? 'Sending...' : 'Send' }}</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    activeConversation: {
      type: Object,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      messages: [],
      messageText: "",
      isSending: false,
    };
  },
  mounted() {
    this.loadMessages();
    this.activeConversation.on("messageAdded", (message) => {
      this.messages.push(message);
      this.scrollToBottom();
    });
  },
  methods: {
    async loadMessages() {
      try {
        const newMessages = await this.activeConversation.getMessages();
        this.messages = newMessages.items;
        this.scrollToBottom();
      } catch (error) {
        console.error("Error loading messages:", error);
      }
    },
    async sendMessage() {
      if (!this.activeConversation) {
        console.error("No active conversation to send message to.");
        return;
      }
      if (!this.messageText) {
        console.error("Message is empty or undefined.");
        return;
      }
      try {
        this.isSending = true;
        await this.activeConversation.sendMessage(this.messageText);

        // Determinar se estamos iniciando ou continuando o fluxo
        const endpoint = this.isFirstMessage ? "trigger-flow" : "continue-flow";
        this.isFirstMessage = false;

        const response = await fetch(`http://localhost:5000/${endpoint}`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            conversationSid: this.activeConversation.sid,
            message: this.messageText,
          }),
        });

        if (!response.ok) {
          throw new Error("Failed to trigger Flow");
        }

        const data = await response.json();
        console.log("Flow triggered successfully:", data);

        this.messageText = "";
      } catch (error) {
        console.error("Error triggering Flow:", error);
      } finally {
        this.isSending = false;
        this.scrollToBottom();
      }
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const container = this.$el.querySelector(".conversation-container");
        container.scrollTop = container.scrollHeight;
      });
    },
  },
};
</script>

<style scoped>
body,
html {
  height: 100%;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f9f9f9;
}

.conversation-container {
  margin: 0 auto;
  max-width: 50%;
  height: 700px;
  padding: 0 20px;
  border: 3px solid #f1f1f1;
  overflow: scroll;
}


.bubble-container {
  display: flex;
  flex-direction: column;
  margin: 10px 0;
}

.bubble {
  display: inline-block;
  border: 2px solid #f1f1f1;
  background-color: #fdfbfa;
  border-radius: 10px;
  padding: 10px;
  max-width: 70%;
  word-wrap: break-word;
}

.myMessage .bubble {
  background-color: #abf1ea;
  border: 2px solid #87E0D7;
  align-self: flex-end;
}

.bubble-container:not(.myMessage) .bubble {
  align-self: flex-start;
}

.name {
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 5px;
}

.message {
  font-size: 14px;
}


.input-container {
  display: inline-block;
  justify-content: center;
  align-content: center;
  border-top: 1px solid #f1f1f1;
  width: 100%;
  box-sizing: border-box;
  margin-top: auto;
  /* Adiciona margem superior automática para ocupar o espaço restante */
}

.input-container input {
  justify-content: center;
  margin-right: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.input-container button {
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

.input-container button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

::-webkit-scrollbar {
  width: 10px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background: #888;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>



<!-- <style scoped>
.conversation-container {
    margin: 0 auto;
    max-width: 50%;
    height: 700px;
    padding: 0 20px;
    border: 3px solid #f1f1f1;
    overflow: scroll;
}

.bubble-container {
    display: flex;
    flex-direction: column;
    margin: 10px 0;
}

.bubble {
    display: inline-block;
    border: 2px solid #f1f1f1;
    background-color: #fdfbfa;
    border-radius: 10px;
    padding: 10px;
    max-width: 70%;
    word-wrap: break-word;
}

.myMessage .bubble {
    background-color: #abf1ea;
    border: 2px solid #87E0D7;
    align-self: flex-end;
}

.bubble-container:not(.myMessage) .bubble {
    align-self: flex-start;
}

.name {
    font-size: 12px;
    font-weight: bold;
    margin-bottom: 5px;
}

.message {
    font-size: 14px;
}

.input-container {
    display:box ;
    justify-content: space-between;
    padding: 10px;
    border-top: 1px solid #f1f1f1;
}

.input-container input {
    flex: 1;
    margin-right: 10px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.input-container button {
    padding: 10px 20px;
    border: none;
    background-color: #007bff;
    color: white;
    border-radius: 5px;
    cursor: pointer;
}

.input-container button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

::-webkit-scrollbar {
    width: 10px;
}

::-webkit-scrollbar-track {
    background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
    background: #888;
}

::-webkit-scrollbar-thumb:hover {
    background: #555;
}
</style> -->

<template>
  <div id="conversation">
    <div class="conversation-container">
      <div v-for="message in messages" :key="message.sid" class="bubble-container"
        :class="{ myMessage: message.author === name }">
        <div class="bubble">
          <div class="name">{{ message.author }}:</div>
          <div class="message">{{ message.body }}</div>
        </div>
      </div>
    </div>
    <div class="input-container">
      <input v-model="messageText" @keyup.enter="sendMessage" placeholder="Enter your message" />
      <button @click="sendMessage" :disabled="isSending">{{ isSending ? 'Sending...' : 'Send' }}</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    activeConversation: {
      type: Object,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      messages: [],
      messageText: "",
      isSending: false,
    };
  },
  mounted() {
    this.loadMessages();
    this.activeConversation.on("messageAdded", (message) => {
      this.messages.push(message);
      this.scrollToBottom();
    });
  },
  methods: {
    async loadMessages() {
      try {
        const newMessages = await this.activeConversation.getMessages();
        this.messages = newMessages.items;
        this.scrollToBottom();
      } catch (error) {
        console.error("Error loading messages:", error);
      }
    },
    async sendMessage() {
      if (!this.activeConversation) {
        console.error("No active conversation to send message to.");
        return;
      }
      if (!this.messageText) {
        console.error("Message is empty or undefined.");
        return;
      }
      try {
        this.isSending = true;
        await this.activeConversation.sendMessage(this.messageText);

        // Determinar se estamos iniciando ou continuando o fluxo
        const endpoint = this.isFirstMessage ? "trigger-flow" : "continue-flow";
        this.isFirstMessage = false;

        const response = await fetch(`http://localhost:5000/${endpoint}`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            conversationSid: this.activeConversation.sid,
            message: this.messageText,
          }),
        });

        if (!response.ok) {
          throw new Error("Failed to trigger Flow");
        }

        const data = await response.json();
        console.log("Flow triggered successfully:", data);

        this.messageText = "";
      } catch (error) {
        console.error("Error triggering Flow:", error);
      } finally {
        this.isSending = false;
        this.scrollToBottom();
      }
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const container = this.$el.querySelector(".conversation-container");
        container.scrollTop = container.scrollHeight;
      });
    },
  },
};
</script>

<style scoped>
body,
html {
  height: 100%;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f9f9f9;
}

.conversation-container {
  margin: 0 auto;
  max-width: 50%;
  height: 700px;
  padding: 0 20px;
  border: 3px solid #f1f1f1;
  overflow: scroll;
}


.bubble-container {
  display: flex;
  flex-direction: column;
  margin: 10px 0;
}

.bubble {
  display: inline-block;
  border: 2px solid #f1f1f1;
  background-color: #fdfbfa;
  border-radius: 10px;
  padding: 10px;
  max-width: 70%;
  word-wrap: break-word;
}

.myMessage .bubble {
  background-color: #abf1ea;
  border: 2px solid #87E0D7;
  align-self: flex-end;
}

.bubble-container:not(.myMessage) .bubble {
  align-self: flex-start;
}

.name {
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 5px;
}

.message {
  font-size: 14px;
}


.input-container {
  display: inline-block;
  justify-content: center;
  align-content: center;
  border-top: 1px solid #f1f1f1;
  width: 100%;
  box-sizing: border-box;
  margin-top: auto;
  /* Adiciona margem superior automática para ocupar o espaço restante */
}

.input-container input {
  justify-content: center;
  margin-right: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.input-container button {
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

.input-container button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

::-webkit-scrollbar {
  width: 10px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background: #888;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>



<!-- <style scoped>
.conversation-container {
    margin: 0 auto;
    max-width: 50%;
    height: 700px;
    padding: 0 20px;
    border: 3px solid #f1f1f1;
    overflow: scroll;
}

.bubble-container {
    display: flex;
    flex-direction: column;
    margin: 10px 0;
}

.bubble {
    display: inline-block;
    border: 2px solid #f1f1f1;
    background-color: #fdfbfa;
    border-radius: 10px;
    padding: 10px;
    max-width: 70%;
    word-wrap: break-word;
}

.myMessage .bubble {
    background-color: #abf1ea;
    border: 2px solid #87E0D7;
    align-self: flex-end;
}

.bubble-container:not(.myMessage) .bubble {
    align-self: flex-start;
}

.name {
    font-size: 12px;
    font-weight: bold;
    margin-bottom: 5px;
}

.message {
    font-size: 14px;
}

.input-container {
    display:box ;
    justify-content: space-between;
    padding: 10px;
    border-top: 1px solid #f1f1f1;
}

.input-container input {
    flex: 1;
    margin-right: 10px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.input-container button {
    padding: 10px 20px;
    border: none;
    background-color: #007bff;
    color: white;
    border-radius: 5px;
    cursor: pointer;
}

.input-container button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

::-webkit-scrollbar {
    width: 10px;
}

::-webkit-scrollbar-track {
    background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
    background: #888;
}

::-webkit-scrollbar-thumb:hover {
    background: #555;
}
</style> -->
