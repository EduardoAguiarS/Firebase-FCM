<script setup>
import HelloWorld from './components/HelloWorld.vue'

// Firebase App
import { initializeApp } from "firebase/app";
import { getMessaging, getToken, onMessage } from "firebase/messaging";

const firebaseConfig = {
  apiKey: "AIzaSyCnjq0dPsCFNBDWyAcdAXY27A-wzVYsom4",
  authDomain: "fcm-web-poc.firebaseapp.com",
  projectId: "fcm-web-poc",
  storageBucket: "fcm-web-poc.appspot.com",
  messagingSenderId: "1064688451347",
  appId: "1:1064688451347:web:82dffb14aa2edb538a24e6"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);

// Messaging
const messaging = getMessaging(app);
onMessage(messaging, (payload) => {
  console.log('Message received. ', payload);
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
  getToken(messaging, { registration, vapidKey: publicVapidKey })
  .then((currentToken) => {
    if (currentToken) {
      console.log("Your token is:", currentToken)
    } else {
      console.log('No registration token available. Request permission to generate one.');
    }
  }).catch((err) => {
    console.log('An error occurred while retrieving token. ', err);
  });
}).catch((err) => {
  console.log('Service worker registration failed, error:', err);
});

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
