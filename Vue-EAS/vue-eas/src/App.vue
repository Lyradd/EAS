<script setup>
import { ref } from 'vue';

const isLoggedIn = ref(false);
const userEmail = ref('');
const userPassword = ref('');
const userUsername = ref('');
const currentUser = ref();
const chatHistory = ref('');
const messageInput = ref('');
const availableChannels = ref();
const selectedChannelId = ref('');
const isChatSelected = ref(false);
const isSignUpMode = ref(false);

async function attemptLogin() {
  try {
    const loginData = {
      email: userEmail.value,
      password: userPassword.value
    };

    const loginReq = await fetch('http://localhost:3000/api/accounts/login', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(loginData),
    });

    if (loginReq.status >= 500) {
      console.log('server error');
    } else if (loginReq.status >= 400) {
      console.log('bad request');
    } else {
      const loginRes = await loginReq.json();

      isLoggedIn.value = true;
      currentUser.value = loginRes.user;

      await fetchUserChannels();

      console.log(loginRes);
      console.log('login success');
    }

  } catch (error) {
    console.log(error);
  }
}

async function performLogout() {
  try {
    const logoutReq = await fetch('http://localhost:3000/api/accounts/logout', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (logoutReq.status >= 500) {
      console.log('server error');
    } else if (logoutReq.status >= 400) {
      console.log('bad request');
    } else {
      isLoggedIn.value = false;
      chatHistory.value = '';
      isChatSelected.value = false;
      console.log('logout success');
    }
  } catch (error) {
    console.log(error);
  }
}

async function fetchUserChannels() {
  try {
    const channelsReq = await fetch('http://localhost:3000/api/channels/', {
      method: 'GET',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (channelsReq.status >= 500) {
      console.log('server error');
    } else if (channelsReq.status >= 400) {
      console.log('bad request');
    } else {
      const channelsRes = await channelsReq.json();
      availableChannels.value = channelsRes.docs;
      console.log(channelsRes);
      console.log('channels fetch success');
    }
  } catch (error) {
    console.log(error);
  }
}

async function fetchSelectedChannel(channelID) {
  try {
    selectedChannelId.value = channelID;
    isChatSelected.value = true;
    const channelReq = await fetch(`http://localhost:3000/api/channels/${channelID}`, {
      method: 'GET',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (channelReq.status >= 500) {
      console.log('server error');
    } else if (channelReq.status >= 400) {
      console.log('bad request');
    } else {
      const channelRes = await channelReq.json();
      chatHistory.value = channelRes.chatHistory;
      console.log(channelRes);
      console.log('channel fetch success');
    }
  } catch (error) {
    console.log(error);
  }
}

async function sendMessage() {
  try {
    const newChatEntry = {
      sender: currentUser.value.username,
      message: messageInput.value,
    };

    const data = {
      chatHistory: [...chatHistory.value, newChatEntry],
    };

    const sendReq = await fetch(`http://localhost:3000/api/channels/${selectedChannelId.value}`, {
      method: 'PATCH',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (sendReq.status >= 500) {
      console.log('server error');
    } else if (sendReq.status >= 400) {
      console.log('bad request');
    } else {
      console.log('send success');
      await fetchSelectedChannel(selectedChannelId.value);
    }

  } catch (error) {
    console.log(error);
  }
}

async function createNewChannel() {
  try {
    const data = {
      authorID: currentUser.value.id,
    };

    const createChannelReq = await fetch(`http://localhost:3000/api/channels/`, {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (createChannelReq.status >= 500) {
      console.log('server error');
    } else if (createChannelReq.status >= 400) {
      console.log('bad request');
    } else {
      console.log('create channel success');
      await fetchUserChannels();
    }

  } catch (error) {
    console.log(error);
  }
}

/*
async function signUpNewUser() {
  try {
    const userData = {
      email: userEmail.value,
      password: userPassword.value,
      username: userUsername.value
    };

    const signUpReq = await fetch('http://localhost:3000/api/accounts/', 
    {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });

    if (signUpReq.status >= 500) {
      console.log('server error');
    } else if (signUpReq.status >= 400) {
      console.log('bad request');
    } else {
      const signUpRes = await signUpReq.json();

      isLoggedIn.value = true;
      currentUser.value = signUpRes.doc;

      await fetchUserChannels();

      console.log(signUpRes);
      console.log('sign up success');
    }
  } catch (error) {
    console.log(error);
  }
}

function toggleSignUpMode() {
  isSignUpMode.value = !isSignUpMode.value;
}
*/

</script>

<template>
  <div class="chat-app">
    <div v-if="!isLoggedIn" class="auth-section">
      <div v-if="!isSignUpMode">
        <h1>EAS</h1>
        <h2>Login</h2>
        <form @submit.prevent="">
          <label for="Email">Email </label>
          <input type="email" id="Email" v-model="userEmail"><br>
          <label for="Password">Password </label>
          <input type="password" id="Password" v-model="userPassword">
        </form>
        <button @click="attemptLogin">Login</button>
        
      </div>
     
    </div>

    <div v-if="isLoggedIn" class="chat-section">
      <h1>Chats</h1>

      <div class="profile-section">
        <h2>Your Profile</h2>
        <p>ID: {{ currentUser.id }}</p>
        <p>Username: {{ currentUser.username }}</p>
        <p>Email: {{ currentUser.email }}</p>
        <button @click="performLogout">Logout</button>
      </div>

      <h2>Available Channels</h2>
      <button @click="createNewChannel" class="action-btn">Create New Channel</button>
      <div v-for="channel in availableChannels" :key="channel.id" class="channel-item">
        <p>Channel {{ channel.id }}</p>
        <button @click="fetchSelectedChannel(channel.id)" class="action-btn">Join</button>
      </div>

      <div v-if="isChatSelected" class="chat-log-section">
        <h2>Chat Log</h2>
        <div v-for="log in chatHistory" :key="log.id" class="chat-message">
          <p>From {{ log.sender }}: {{ log.message }}</p>
        </div>
        <input type="text" v-model="messageInput" placeholder="Type your message...">
        <button @click="sendMessage" class="action-btn">Send</button>
      </div>
    </div>
  </div>



  
</template>

<style scoped>
.chat-app {
  font-family: 'Arial', sans-serif;
  max-width: 800px;
  margin: auto;
  padding: 20px;
}

.auth-section,
.chat-section {
  background-color: #f8f8f8;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 8px;
  color: #000; /* Set text color to solid black */
}

h1 {
  font-size: 24px;
  margin-bottom: 20px;
}

form {
  margin-top: 10px;
}

input {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #3498db;
  color: #fff;
  border: none;
  padding: 10px;
  cursor: pointer;
  border-radius: 4px;
}

button:hover {
  background-color: #2980b9;
}

.profile-section,
.chat-log-section {
  margin-top: 20px;
}

.action-btn {
  background-color: #2ecc71;
}

.action-btn:hover {
  background-color: #27ae60;
}

.channel-item {
  margin-bottom: 10px;
}

.chat-message {
  margin-bottom: 5px;
}
</style>

