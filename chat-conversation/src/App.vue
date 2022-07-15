<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <ChatConversation/>
</template>

<script>
import ChatConversation from './components/ChatConversation.vue'

export default {
  name: 'App',
  components: {
    ChatConversation
}
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
