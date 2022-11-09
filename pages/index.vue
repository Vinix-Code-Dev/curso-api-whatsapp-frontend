<template>
  <v-row>
    <!-- Chat list -->
    <v-col sm="3">
      <v-card elevation="7" class="chat-list">
        <v-list class="py-0" two-line>
          <v-list-item-group v-model="selected" active-class="green--text">
            <template v-for="(item, index) in items">
              <v-list-item :key="item.title">
                <template>
                  <v-list-item-content>
                    <v-list-item-title v-text="item.title"></v-list-item-title>

                    <v-list-item-subtitle
                      v-text="item.subtitle"
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
export default {
  data: () => ({
    selected: [2],
    items: [
      {
        subtitle: `I'll be in your neighborhood doing errands this weekend. Do you want to hang out?`,
        title: 'Ali Connors',
      },
      {
        subtitle: `Wish I could come, but I'm out of town this weekend.`,
        title: 'Jennifer',
      },
      {
        subtitle: 'Do you have Paris recommendations? Have you ever been?',
        title: 'Sandra Adams',
      },
      {
        subtitle:
          'Have any ideas about what we should get Heidi for her birthday?',
        title: 'Trevor Hansen',
      },
      {
        subtitle:
          'We should eat this: Grate, Squash, Corn, and tomatillo Tacos.',
        title: 'Britta Holt',
      },
      {
        subtitle: `I'll be in your neighborhood doing errands this weekend. Do you want to hang out?`,
        title: 'Ali Connors',
      },
      {
        subtitle: `Wish I could come, but I'm out of town this weekend.`,
        title: 'Jennifer',
      },
      {
        subtitle: 'Do you have Paris recommendations? Have you ever been?',
        title: 'Sandra Adams',
      },
      {
        subtitle:
          'Have any ideas about what we should get Heidi for her birthday?',
        title: 'Trevor Hansen',
      },
      {
        subtitle:
          'We should eat this: Grate, Squash, Corn, and tomatillo Tacos.',
        title: 'Britta Holt',
      },
    ],
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
  }),
  methods: {
    sendMessage() {
      this.messages.push({
        outgoing: true,
        body: this.message,
        created_at: this.$moment().format('L'),
        status: 'sent',
      })
      this.message = ''
    },
    formatReadStatus(status) {
      const statuses = {
        read: {
          color: 'blue',
          icon: 'mdi-check-all',
          // tooltip: $t('chat.msgDelivered'),
          tooltip: 'Read',
        },
        delivered: {
          color: 'grey',
          icon: 'mdi-check-all',
          // tooltip: $t('chat.msgRead'),
          tooltip: 'Delivered',
        },
        sent: {
          color: 'grey',
          icon: 'mdi-check',
          // tooltip: $t('chat.msgSent'),
          tooltip: 'Sent',
        },
        failed: {
          color: 'red',
          icon: 'mdi-alert-circle',
          // tooltip: $t('chat.msgFailed'),
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