<script>
import { onUpdated, ref } from 'vue';

export default {
  props:['chatrooms', 'username'],
  data() {
    return {
      isTyping: ref(null),
      chatSocket: new WebSocket(
                'wss://2391-79-118-244-81.ngrok.io/ws/'
                + this.$route.params.room
                + '/'
            ),
      text: ref(''),
      dense: ref(false),
      messages: ref([])
    }
  },
  computed: {
    myUsername: {
      get: function () {
        return this.username;
      },
      set: function (user) {
        this.$emit("setUsername", user);
      },
    }
  },
  methods:{
    async fetchMessages() {
      try {
        let roomname = this.$route.params.room;
        if(roomname){
          let chatRoom = this.chatrooms.filter(room => room.slug == roomname)[0];
          this.messages = [];
          const res = await fetch(`https://2391-79-118-244-81.ngrok.io/api/messagesbyroom/?room=${chatRoom.id}`);
          this.messages = await res.json();
        }

      } catch (err) {
        this.fetchError = err;
      }
    },
    scrollToBottom() {

      let objDiv = document.getElementById("chat-messages");
      objDiv.scrollTop = objDiv.scrollHeight;
    },

    iAmTyping(){
      this.chatSocket.send(JSON.stringify({
            'message': '...',
            'username': this.username,
            'room': this.$route.params.room
        }));
    

    },
    sendMessage(){
      this.chatSocket.send(JSON.stringify({
            'message': this.text,
            'username': this.username,
            'room': this.$route.params.room
        }));
      
      this.text = "";
    },
   
  },
  mounted(){
    this.fetchMessages(); 
    this.chatSocket.onmessage = (e)=> {

        const data = JSON.parse(e.data);
        if (data.message && data.message != '...') {
          this.messages.push(data)
 
        } else if(data.message && data.message == '...' && data.username != this.myUsername){
            this.isTyping = data.username
            setTimeout(()=>this.isTyping=null, 1000)
        } else if (data.message == '') {
            alert('The message is empty!');
        }

      };
    
  
  },
  updated(){
    this.scrollToBottom();
  }
}
</script>

<template>
  <div>
      <div id="chat-messages" style="height:65vh; width: 100%; overflow-y: scroll;" class="q-pa-md justify-center">
            <q-chat-message v-for="message in messages" :key="message.id"
              :name="message.username"
              :text="[message.content]"
              :sent="username == message.username"
            />
      </div>

    <div class="row justify-start">  
      <q-chat-message 
        :style="{visibility: isTyping ? 'visible' : 'hidden'}"
        :name="isTyping ? isTyping : 'Nobody'"
        class="col-12" 
        bg-color="orange"
      >
        <q-spinner-dots size="2rem" />
      </q-chat-message>
    <q-input class="col-xs-6 col-sm-4 col-md-2" 
      bottom-slots 
      v-model="myUsername"
      outlined 
      label="Username" 
      :dense="dense" 
      >
      <template v-slot:append>
            <q-icon v-if="username !== ''" name="close" @click="myUsername='Anonymous'" class="cursor-pointer" />
      </template>
    </q-input>
    <q-input 
      class="col-xs-12 col-sm-12 col-md-10" 
      outlined 
      bottom-slots 
      v-model="text" 
      label="Message" 
      :dense="dense"
      @keyup="iAmTyping"
    >
          <template v-slot:append>
            <q-icon v-if="text !== ''" name="close" @click="text = ''" class="cursor-pointer" />
          </template>
          <template v-slot:after>
            <q-btn @click="sendMessage" round dense flat icon="send" />
          </template>
        </q-input>
    </div>
  </div>
</template>