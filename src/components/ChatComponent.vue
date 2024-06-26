<template>
    <div id="chat">
      <h1>
        Welcome to the Vue chat app<span v-if="nameRegistered">, {{ name }}</span>!
      </h1>
      <p>{{ statusString }}</p>
      <div v-if="!nameRegistered">
        <input @keyup.enter="registerName" v-model="name" placeholder="Enter your name" />
        <button @click="registerName">Register name</button>
      </div>
      <div v-if="nameRegistered && !activeConversation && isConnected">
        <button @click="createOrJoinConversation">Join chat</button>
      </div>
      <ConversationComponent
        v-if="activeConversation"
        :active-conversation="activeConversation"
        :name="name"
      />
    </div>
  </template>
  
  <script>
  import { Client as ConversationsClient } from "@twilio/conversations";
  import ConversationComponent from "./ConversationComponent.vue";
  
  export default {
    components: { ConversationComponent },
    data() {
      return {
        statusString: "",
        activeConversation: null,
        name: "",
        nameRegistered: false,
        isConnected: false,
      };
    },
    methods: {
      async initConversationsClient() {
        try {
          this.statusString = "Connecting to Twilio…";
          const token = await this.getToken(this.name);
          this.conversationsClient = new ConversationsClient(token);
  
          this.conversationsClient.on("connectionStateChanged", (state) => {
            switch (state) {
              case "connected":
                this.statusString = "You are connected.";
                this.isConnected = true;
                break;
              case "disconnecting":
                this.statusString = "Disconnecting from Twilio…";
                break;
              case "disconnected":
                this.statusString = "Disconnected.";
                this.isConnected = false;
                break;
              case "denied":
                this.statusString = "Failed to connect.";
                this.isConnected = false;
                break;
              default:
                this.statusString = `Unknown connection state: ${state}`;
            }
          });
        } catch (error) {
          console.error("Error initializing Twilio Conversations client:", error);
        }
      },
  
      async registerName() {
        try {
          this.nameRegistered = true;
          await this.initConversationsClient();
        } catch (error) {
          console.error("Error registering name:", error);
        }
      },
  
      async createOrJoinConversation() {
        try {
          let conversation = await this.conversationsClient.getConversationByUniqueName(this.name);
          this.activeConversation = conversation;
          await this.addParticipantToConversation(conversation.sid, "AGWeb");
          await this.addParticipantToConversation(conversation.sid, this.name);
        } catch (error) {
          if (error.message.includes("Not Found")) {
            try {
              let newConversation = await this.conversationsClient.createConversation({
                uniqueName: this.name,
              });
              await newConversation.join();
              this.activeConversation = newConversation;
              await this.addParticipantToConversation(newConversation.sid, "AGWeb");
              await this.addParticipantToConversation(newConversation.sid, this.name);
            } catch (createError) {
              console.error("Error creating conversation:", createError);
            }
          } else {
            console.error("Error fetching conversation:", error);
          }
        }
  
        if (!this.activeConversation) {
          console.error("activeConversation not properly initialized.");
        }
      },
  
      async addParticipantToConversation(conversationSid, participantIdentity) {
        try {
          const conversation = await this.conversationsClient.getConversationBySid(conversationSid);
  
          const participants = await conversation.getParticipants();
          const participantExists = participants.some(
            (participant) => participant.identity === participantIdentity
          );
  
          if (!participantExists) {
            await conversation.add(participantIdentity);
          } else {
            console.log(`Participant ${participantIdentity} is already in conversation ${conversationSid}.`);
          }
        } catch (error) {
          console.error(`Error adding participant ${participantIdentity} to conversation ${conversationSid}:`, error);
        }
      },
  
      async getToken(identity) {
        try {
          const response = await fetch(`http://localhost:5000/auth/user/${identity}`);
          if (!response.ok) {
            throw new Error("Error fetching Twilio authentication token");
          }
          const responseJson = await response.json();
          return responseJson.token;
        } catch (error) {
          console.error("Error fetching Twilio authentication token:", error);
          throw error;
        }
      },
    },
  };
  </script>
  

<style scoped>
ul {
    list-style-type: none;
    padding: 0;
}

li {
    display: inline-block;
    margin: 0 10px;
}

a {
    color: #42b983;
}
</style>
