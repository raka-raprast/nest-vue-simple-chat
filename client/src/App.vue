<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

const socket = io("http://localhost:3001");

const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const roomId = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  socket.on("message", (message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value, roomId: roomId.value }, () => {
    // Joining the general room
    joined.value = true;
    // Joining the specific room
    socket.emit("joinRoom", roomId.value);
  });
  socket.emit("findAllMessages", { roomId: roomId.value }, (response) => {
    messages.value = response;
  });
};
const sendMessage = () => {
  // console.log(roomId.value);
  socket.emit(
    "createMessage",
    { text: messageText.value, roomId: roomId.value },
    () => {
      messageText.value = "";
    }
  );
};

// const sendMessage = () => {
//   socket.emit(
//     "createMessage",
//     { text: messageText.value, roomId: roomId.value },
//     () => {
//       // Manually append the sent message to the message list
//       messages.value.push({ name: name.value, text: messageText.value });
//       messageText.value = ""; // Clear the message input
//     }
//   );
// };

let timeout;

const emitTyping = () => {
  socket.emit("typing", { isTyping: true, roomId: roomId.value });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false, roomId: roomId.value });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label>What's your name?</label>
        <input v-model="name" />
        <label>Room id</label>
        <input v-model="roomId" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">
          [{{ message.name }}] : {{ message.text }}
        </div>
      </div>
      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat {
  padding: 20px;
  height: 100vh;
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.message-container {
  flex: 1;
}
</style>
