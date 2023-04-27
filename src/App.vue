<script setup>
import HelloWorld from './components/HelloWorld.vue'

// Firebase App
import { initializeApp } from "firebase/app";
import { getMessaging, getToken, onMessage, deleteToken} from "firebase/messaging";
import { ref } from 'vue'

const firebaseConfig = {
  apiKey: "AIzaSyCnjq0dPsCFNBDWyAcdAXY27A-wzVYsom4",
  authDomain: "fcm-web-poc.firebaseapp.com",
  projectId: "fcm-web-poc",
  storageBucket: "fcm-web-poc.appspot.com",
  messagingSenderId: "1064688451347",
  appId: "1:1064688451347:web:82dffb14aa2edb538a24e6"
};

const token = ref(null);
const register = ref(null);
const buttonValue = ref('notification')

// Initialize Firebase
const app = initializeApp(firebaseConfig);

// Messaging
const messaging = getMessaging(app);
onMessage(messaging, (payload) => {
  const notificationTitle = payload.data.title;
  const notificationOptions = {
    body: payload.data.body,
    icon: payload.data.icon,
  };

  // Show a notification
  new Notification(notificationTitle, notificationOptions);
});

const publicVapidKey = "BDBTe05KkWqb5YST2u2Ghpt2s-ZOa-I6xEnOWDXSh0A1exqgxbd5--XNSCwNCnRBs3SsnNcxwc-HTk4ys-VH01I"

navigator.serviceWorker.register('/firebase-messaging-sw.js')
.then((registration) => {
  register.value = registration
}).catch((err) => {
  console.log('Service Worker registration failed: ', err);
});

const reg = register.value

if (window.localStorage.getItem('control') === null) {
  window.localStorage.setItem('control', true);
} else {
  if (window.localStorage.getItem('control') === 'true') {
    buttonValue.value = 'notification'
  } else {
    buttonValue.value = 'disable notification'
  }
}

function permission() {
  if (window.localStorage.getItem('control') === 'true') {
    Notification.requestPermission().then((permission) => {
      if (permission === 'granted') {
        getToken(messaging, { reg, vapidKey: publicVapidKey })
        .then((currentToken) => {
          if (currentToken) {
            token.value = currentToken
            buttonValue.value = 'disable notification'
            window.localStorage.setItem('control', false);
          } else {
            console.log('No registration token available. Request permission to generate one.');
          }
        }).catch((err) => {
          console.log('An error occurred while retrieving token. ', err);
        });
      } else {
        console.log('Unable to get permission to notify.');
      }
    });
  } else {
    deleteToken(messaging, token.value)
    .then(() => {
      console.log('Token deleted.');
      token.value = null
      buttonValue.value = 'notification'
      window.localStorage.setItem('control', true);
    }).catch((err) => {
      console.log('Unable to delete token. ', err);
    });
  }
}
</script>

<template>
  <div>
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <div>
    <button v-on:click="permission">{{ buttonValue }}</button>
    <p>{{ token }}</p>
  </div>
  <HelloWorld msg="Vite + Vue" />
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
