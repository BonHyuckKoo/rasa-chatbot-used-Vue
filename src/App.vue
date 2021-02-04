<template>
  <v-app>
      <ChatRoom />
  </v-app>
</template>

<script>
import ChatRoom from './components/ChatRoom';

export default {
  name: 'App',

  components: {
    ChatRoom
  },

  data: () => ({
    
  }),
};
</script>
