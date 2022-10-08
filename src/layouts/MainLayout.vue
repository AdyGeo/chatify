
<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg">
          </q-avatar>
          Chatty
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>

      <div>
        <q-splitter v-model="splitterModel" style="height: 90vh">

          <template v-slot:before>
            <q-tabs v-model="tab" vertical class="text-primary">
              <q-route-tab v-for="chatroom in chatrooms" :key="chatroom.id" :to="{name:'room', params:{room: chatroom.slug}}" :name="chatroom.slug" :label="chatroom.slug" />
            </q-tabs>
          </template>
          <template v-slot:after>
        <q-tab-panels
          v-model="tab"
          animated
          swipeable
          vertical
          transition-prev="jump-up"
          transition-next="jump-up"
        >
          <q-tab-panel :name="$route.params.room">
            <router-view :chatrooms="chatrooms" :username="username" @setUsername="(e) => this.username = e"/>
          </q-tab-panel>
        </q-tab-panels>
      </template>
        </q-splitter>
      </div>

    </q-page-container>
  </q-layout>
</template>


<script>
import { ref } from 'vue'


export default {

  setup() {
    return {
      username: ref("Anonymous"),
      tab: ref("chatrooms"),
      chatrooms: ref([]),
      splitterModel: ref(20)
    }
  },
  methods:{
    async fetchChatrooms() {
      try {
        this.chatrooms = [];
        const res = await fetch("http://localhost:8000/api/chatrooms/");
        this.chatrooms = await res.json();
      } catch (err) {
        this.fetchError = err;
      }
    },
    usernameHasChanged(username){
      this.username=username;
    }
  },
  mounted(){
    this.fetchChatrooms();
  }
}
</script>
