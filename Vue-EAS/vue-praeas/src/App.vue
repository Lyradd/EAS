<template>
  <div>
    <h1>Pra EAS</h1>

    <div v-if="isUserLogged">
      <button @click="performLogout">Logout</button>
    </div>

    <br /><br />

    <div v-if="!isUserLogged">
      <h3>{{ isSignUpMode ? 'SIGN UP' : 'LOGIN' }}</h3>
      <label for="email">Email :</label>
      <input type="email" v-model="emailInput" /><br />
      <label for="password">Password :</label>
      <input type="password" v-model="passwordInput" /><br />

      <div v-if="isSignUpMode">
        <label for="username">Username :</label>
        <input type="text" v-model="usernameInput" /><br />
      </div>

      <button @click="isSignUpMode ? signUpNewUser : attemptLogin">
        {{ isSignUpMode ? 'Sign Up' : 'Login' }}
      </button>

      <p>{{ loginMessage }}</p>

      <button @click="toggleSignUpMode">
        {{ isSignUpMode ? 'Back to Login' : 'Sign Up Instead' }}
      </button>
    </div>

    <div v-if="isUserLogged">
      <div>
        <h3>CREATE CHANNELS</h3>
        <input type="name_channel" v-model="name_channel" />
        <button @click="createChannel()">Channels</button>
        <p>{{ createMessage }}</p>
      </div>
      <br />

      <div v-if="!joined">
        <h3>JOIN CHANNEL</h3>
        <button @click="getChannels()">Get Channels</button>
        <div v-for="channel in channels" :key="channel.id">
          <p>Nama : {{ channel.channel }}</p>
          <button @click="joinChannel(channel.id)">join</button>
        </div>
      </div>

      <br />

      <div v-if="joined">
        <h3>SEND MESSAGE</h3>
        <label for="message">Message </label>
        <input type="text" v-model="message" />
        <button @click="sendMessage()">Send</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const isUserLogged = ref(false);
const emailInput = ref('');
const passwordInput = ref('');
const usernameInput = ref('');
const currentUser = ref();
const chatHistory = ref('');
const messageInput = ref('');
const availableChannels = ref();
const selectedChannelId = ref('');
const isChatSelected = ref(false);
const isSignUpMode = ref(false);
const loginMessage = ref('');
const channels = ref('');
const name_channel = ref('');
const createMessage = ref('');
const join_channel = ref('');
const user = ref('');
const message = ref('');
const idChannel = ref('');
const chatLog = ref('');
const username = ref('');
const singed = ref(false);
const email_sign = ref('');
const password_sign = ref('');
const signMessage = ref('');
const joined = ref(false);

async function attemptLogin() {
  try {
    const req = await fetch('http://localhost:3000/api/accounts/login', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: emailInput.value,
        password: passwordInput.value,
      }),
    });
    const data = await req.json();
    console.log(data);

    loginMessage.value = data.message;
    user.value = data.user;
    isUserLogged.value = true;
    singed.value = false;
    joined.value = false;
    console.log(user.value);
    console.log(user.value.username);
  } catch (err) {
    console.log(err);
  }
}

async function getChannels() {
  try {
    const req = await fetch('http://localhost:3000/api/channels/');
    const data = await req.json();
    console.log(data);

    channels.value = data.docs;
    console.log(channels.value);
  } catch (err) {
    console.log(err);
  }
}

async function createChannel() {
  try {
    const req = await fetch('http://localhost:3000/api/channels/', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        channel: name_channel.value,
      }),
    });
    const data = await req.json();
    console.log(data);
    createMessage.value = data.message;
  } catch (err) {
    console.log(err);
  }
}

async function sendMessage() {
  try {
    const newChat = {
      sender: user.value.usename,
      message: message.value,
    };

    const newdata = {
      chathistory: [...chatLog.value, newChat],
    };

    const req = await fetch(`http://localhost:3000/api/channels/${idChannel.value}`, {
      method: 'PATCH',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newdata),
    });
    const data = await req.json();
    console.log(newdata);
  } catch (err) {
    console.log(err);
  }
}

async function joinChannel(channelID) {
  try {
    idChannel.value = channelID;
    const req = await fetch(`http://localhost:3000/api/channels/${channelID}`);
    const data = await req.json();
    console.log(data);

    chatLog.value = data.chathistory;
    console.log(chatLog.value);
    console.log(idChannel.value);
    joined.value = true;
  } catch (err) {
    console.log(err);
  }
}

async function signIN() {
  try {
    const req = await fetch('http://localhost:3000/api/accounts', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: email_sign.value,
        password: password_sign.value,
        username: username.value,
      }),
    });
    const data = await req.json();
    console.log(data);
    signMessage.value = data.message;
  } catch (err) {
    console.log(err);
  }
}

function isSignIn() {
  singed.value = true;
  isUserLogged.value = false;
}

async function Logout() {
  try {
    const req = await fetch('http://localhost:3000/api/accounts/logout', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });
    const data = await req.json();
    console.log(data);
    isUserLogged.value = false;
  } catch (err) {
    console.log(err);
  }
}
</script>

<style scoped>
/* ... Your existing styles ... */
</style>
