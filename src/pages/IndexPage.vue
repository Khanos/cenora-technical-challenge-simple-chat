<template>
<q-page-container id="chat-container">
  <q-page class="flex flex-column flex-center">
    <img
      alt="Quasar logo"
      src="~assets/quasar-logo-vertical.svg"
      style="width: 200px; height: 200px"
    />
    <h6>Hello Quasar From Docker</h6>
    <q-separator />
    <div class="row">
      <div class="col-12">
        <q-list dense bordered padding class="rounded-borders">
          <q-item v-ripple v-for="(item, index) in chatMessages"
        :key="index">
            <q-item-section v-if="!item.init">
              {{item.user}} says: {{item.text}}
            </q-item-section>
            <q-item-section v-else>
              {{item.text}}
            </q-item-section>
          </q-item>
        </q-list>
      </div>
    </div>
    <div class="row">
      <div class="col-10">
        <q-input v-model="text" label="Standard"/>
      </div>
      <div class="col-2">
        <q-btn round color="secondary" icon="send" @click="sendMessage" />
      </div>
    </div>
  </q-page>
</q-page-container>
</template>

<script>
import { defineComponent } from "vue";
const { DeepstreamClient } = window.DeepstreamClient;
const client = new DeepstreamClient('localhost:6020')
client.login();

export default defineComponent({
  name: "IndexPage",
  data(){
    return {
      init: false,
      client: client,
      text: '',
      chatMessages: [],
      user: `user-${Math.floor(Math.random() * 100)}`,
    }
  },
  mounted(){
    if (!this.init) {
      this.init = true;
      this.client.event.subscribe('messages', function(value){
        const message = JSON.parse(value);
        this.chatMessages.push({text: message.text, user: message.user, init: message.init});
      }.bind(this));
    }
    if (this.chatMessages.length === 0) {
      this.sendMessage();
    }
  },
  methods: {
    sendMessage(){
      if (this.chatMessages.length === 0) {
        this.client.event.emit('messages', JSON.stringify({text: `Welcome to the chat user: ${this.user}!`, user: this.user, init: true}));
        } else {
          if(this.text.length === 0) return
          this.client.event.emit('messages', JSON.stringify({text: this.text, user: this.user, init: false}));
      }
      this.text = ''
    },
  }
});
</script>
<style>
.flex-column {
  flex-direction: column;
}
#chat-container{
  max-width: 600px;
}
</style>
