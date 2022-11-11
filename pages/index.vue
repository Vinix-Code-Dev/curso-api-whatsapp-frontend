<template>
  <v-row>
    <!-- Chat list -->
    <v-col sm="3">
      <v-card elevation="7" class="chat-list">
        <v-card class="pa-2" color="#424242" dark>Chats</v-card>
        <v-list class="py-0" two-line>
          <v-list-item-group v-model="selected" active-class="green--text">
            <template v-for="(item, index) in items">
              <v-list-item :key="item.wa_id" @click="loadConversation(item)">
                <template>
                  <v-list-item-content>
                    <v-list-item-title v-text="item.wa_id"></v-list-item-title>

                    <v-list-item-subtitle
                      v-text="item.body"
                    ></v-list-item-subtitle>
                  </v-list-item-content>
                </template>
              </v-list-item>

              <v-divider
                v-if="index < items.length - 1"
                :key="index"
              ></v-divider>
            </template>
          </v-list-item-group>
        </v-list>
      </v-card>
    </v-col>

    <!-- Conversations -->
    <v-col md="9">
      <v-card elevation="7">
        <v-card class="chat-box wpp-bg" color="#efeae2">
          <v-card class="pa-2" color="#424242" dark>Ali Connors</v-card>
          <div
            v-for="(message, index) in messages"
            :key="index"
            :class="[
              'pa-1',
              'd-flex',
              { 'justify-start': !message.outgoing },
              { 'justify-end': message.outgoing },
            ]"
          >
            <v-card
              class="pa-2 ma-2 message-card"
              :color="message.outgoing ? 'green accent-1' : 'white'"
            >
              <div>
                <div>{{ message.body }}</div>
                <p class="text-right text-subtitle-2 font-italic">
                  {{ message.created_at }}
                  <v-icon
                    v-bind="attrs"
                    :color="formatReadStatus(message.status).color"
                    small
                    v-on="on"
                    >{{ formatReadStatus(message.status).icon }}</v-icon
                  >
                </p>
              </div>
            </v-card>
          </div>
        </v-card>
        <v-divider></v-divider>
        <div class="pa-3">
          <v-text-field
            v-model="message"
            dense
            outlined
            background-color="white"
            :append-outer-icon="'mdi-send'"
            filled
            clear-icon="mdi-close-circle"
            clearable
            label="Type a message..."
            type="text"
            class="message-box"
            @click:append-outer="sendMessage"
            @keydown.enter.exact.prevent="sendMessage"
          ></v-text-field>
        </div>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import Vue from 'vue'
import Echo from 'laravel-echo'
window.Pusher = require('pusher-js')

export default {
  data: () => ({
    selected: [2],
    items: [],
    messages: [
      {
        outgoing: false,
        body: 'Nunc tellus magna, volutpat vel orci eu, venenatis vestibulum magna. Morbi fermentum, purus laoreet egestas maximus, metus ex bibendum nulla, in posuere nunc neque id nulla.',
        created_at: '05/02/2022',
        status: 'sent',
      },
      {
        outgoing: true,
        body: 'estibulum pellentesque maximus lacus, quis viverra justo pharetra sed. Curabitur tempus consequat dolor, ut gravida dui pulvinar eget.',
        created_at: '05/02/2022',
        status: 'read',
      },
      {
        outgoing: false,
        body: 'Nulla id eros consequat purus interdum iaculis quis ut orci. Mauris dapibus turpis sit amet egestas consectetur. ',
        created_at: '05/02/2022',
        status: 'sent',
      },
      {
        outgoing: true,
        body: 'estibulum pellentesque maximus lacus, quis viverra justo pharetra sed. Curabitur tempus consequat dolor, ut gravida dui pulvinar eget.',
        created_at: '05/02/2022',
        status: 'delivered',
      },
    ],
    message: '',
    selectedChat: {},
  }),
  created() {
    this.loadMessages()
  },
  mounted() {
    window.Echo = new Echo({
      broadcaster: 'pusher',
      key: process.env.pusherAppKey,
      cluster: process.env.pusherAppCluster,
      forceTLS: true,
    })

    window.Echo.channel('webhooks').listen('Webhook', (res) => {
      const message = res?.message
      const changed = res?.change

      if (this.selectedChat?.wa_id === message.wa_id) {
        if (changed === false) {
          this.appendMessage(message)
          this.scrollToBottom()
        } else {
          const msgIndex = this.messages?.findIndex((el) => {
            console.log('Element: ' + el.wam_id)
            console.log('Searched: ' + message.wam_id)
            return el.wam_id === message.wam_id
          })

          if (msgIndex !== -1) {
            Vue.set(this.messages, msgIndex, message)
          }
        }
      }
    })
  },
  methods: {
    appendMessage(message) {
      this.messages = this.messages.concat(message)
    },
    loadMessages() {
      this.$axios.get('/messages').then(({ data }) => {
        this.items = data.data
      })
    },
    loadConversation(chat) {
      this.selectedChat = chat
      this.$axios.get('/messages/' + chat.wa_id).then(({ data }) => {
        this.messages = data.data
        this.scrollToBottom()
      })
    },
    sendMessage() {
      const payload = {
        wa_id: this.selectedChat.wa_id,
        body: this.message,
      }
      this.$axios
        .post('/messages', payload)
        .then(({ data }) => {
          this.messages.push({
            outgoing: data.data.outgoing,
            body: data.data.body,
            created_at: this.$moment(data.data.created_at).format('L'),
            status: data.data.status,
            wam_id: data.data.wam_id,
          })
          this.message = ''
          this.scrollToBottom()
        })
        .catch((err) => {
          console.log(err)
        })
    },
    scrollToBottom() {
      setTimeout(() => {
        const container = this.$el.querySelector('.chat-box')
        if (container?.scrollHeight) {
          container.scrollTop = container?.scrollHeight
        }
      }, 100)
    },
    formatReadStatus(status) {
      const statuses = {
        read: {
          color: 'blue',
          icon: 'mdi-check-all',
          tooltip: 'Read',
        },
        delivered: {
          color: 'grey',
          icon: 'mdi-check-all',
          tooltip: 'Delivered',
        },
        sent: {
          color: 'grey',
          icon: 'mdi-check',
          tooltip: 'Sent',
        },
        failed: {
          color: 'red',
          icon: 'mdi-alert-circle',
          tooltip: 'Failed',
        },
      }

      if (statuses?.[status]) {
        return statuses[status]
      } else {
        return {
          color: 'grey',
          icon: 'mdi-cards-diamond-outline',
          tooltip: '',
        }
      }
    },
  },
}
</script>

<style>
.message-box .v-text-field__details {
  display: none;
}
.chat-box {
  height: 80vh;
  overflow-y: scroll;
}
.message-card {
  max-width: 60%;
}
.chat-list {
  height: 85vh;
}
.wpp-bg {
  background-image: url('/bg-whatsapp.png');
  background-repeat: repeat;
  border-color: #efeae2;
}
</style>