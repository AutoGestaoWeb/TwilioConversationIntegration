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
      <div class="input-container">
        <textarea v-model="messageText" @keyup.enter="sendMessage" class="form-control"
          placeholder="Digite sua mensagem" /> <!-- Modificado para ocupar toda a largura -->
        <button @click="sendMessage" :disabled="isSending" class="btn btn-success">
          <i class="bi bi-send"></i>
        </button>
      </div>
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
      isFirstMessage: true,
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
            name: this.name
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

<style>
body,
html {
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}

.conversation-container {
  margin: 0 auto;
  width: 100%;
  height: 800px;
  padding: 0 20px;
  overflow: auto;

  /* Mudança de scroll para auto */
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
  max-width: 40%;
  word-wrap: break-word;
}

.myMessage .bubble {
  background-color: #008069;
  border: 1px solid #6666667c;
  align-self: flex-end;
  color: rgb(255, 255, 255);
  text-align: start;
}

.bubble-container:not(.myMessage) .bubble {
  background-color: rgb(253, 253, 253);
  border: 1px solid #49494996;
  align-self: flex-start;
  color: rgb(0, 0, 0);
  text-align: start;
}

.name {
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 5px;
  text-align: left;
}

.message {
  font-size: 14px;
}

.input-container {
  display: flex;
  border-top: 1px solid #f1f1f1;
  background-color: #f9f9f9;
  padding: 10px;
  box-sizing: border-box;
  width: 100%;
  position: sticky;
  bottom: 0;
  left: 0;
  background: #fff;
}


/* Estilo para o input de mensagem */
.input-container textarea {
  flex: 1;
  height: auto;
  min-height: 20px;
  max-height: 150px;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  word-wrap: break-word;
  /* Quebrar as palavras longas */
}


/* Estilo para o botão de enviar */
.input-container button {
  padding: 10px 20px;
  border: none;
  background-color: rgb(167, 219, 46);
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

.input-container button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

/* Estilizar a barra de rolagem para afastá-la da conversa */
::-webkit-scrollbar {
  width: 12px;
  /* Largura do scrollbar */
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  margin: 20px 0;
  /* Adiciona margem para afastar o scroll */
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 6px;
  border: 3px solid transparent;
  /* Aumenta a largura e cria espaço ao redor do thumb */
  background-clip: padding-box;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}

/* Mobile */
@media (max-width: 600px) {
  .bubble {
    max-width: 70%;
  }

  .input-container textarea {
    font-size: 14px;
  }

  .input-container button {
    padding: 10px;
    font-size: 14px;
  }
}

/* Tablets */
@media (min-width: 601px) and (max-width: 1024px) {
  .bubble {
    max-width: 60%;
  }

  .input-container textarea {
    font-size: 16px;
  }

  .input-container button {
    padding: 12px 16px;
    font-size: 16px;
  }
}
</style>