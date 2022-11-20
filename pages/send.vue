<template>
  <v-row>
    <v-col>
      <v-card elevation="7">
        <v-card class="pa-2" color="#424242" dark>Send Messages</v-card>
        <div class="pa-4">
          <v-select
            v-model="item"
            :items="items"
            label="Templates"
            outlined
            @change="selectTemplate"
          ></v-select>

          <template v-if="template">
            <v-divider class="mb-4"></v-divider>
            <v-alert v-if="template.status !== 'APPROVED'" type="error"
              >Template unapproved and can't be used to send messages.</v-alert
            >

            <div class="my-5">
              <h5 class="text-h5">Recipients</h5>
              <v-textarea
                v-model="recipients"
                outlined
                hint="Enter one recipient per line."
                persistent-hint
              >
              </v-textarea>
            </div>

            <div v-if="template.header" class="my-5">
              <h5 class="text-h5">Header</h5>
              <p v-if="template.header.format == 'TEXT'">
                {{ template.header.text }}
              </p>
              <v-text-field
                v-else
                v-model="header_url"
                outlined
                class="mt-2"
                :label="`${template.header.format} URL`"
              ></v-text-field>
            </div>

            <div v-if="template.body" class="my-5">
              <h5 class="text-h5">Body</h5>
              <p class="pre-wrap">{{ template.body }}</p>
              <v-text-field
                v-for="(placeholder, index) in template.body_placeholders"
                :key="index"
                v-model="body_placeholders[index]"
                outlined
                :label="placeholder.text"
              ></v-text-field>
            </div>

            <div v-if="template.footer" class="my-5">
              <h5 class="text-h5">Footer</h5>
              <p class="pre-wrap">{{ template.footer }}</p>
            </div>

            <div v-if="template.buttons" class="my-3">
              <h5 class="text-h5">Buttons</h5>
              <ul>
                <li v-for="(button, index) in template.buttons" :key="index">
                  {{ button.text }} <em>({{ button.type }})</em>
                </li>
              </ul>
            </div>

            <div class="mt-8">
              <v-btn
                block
                large
                color="teal darken-1"
                dark
                :loading="sending"
                @click="send"
                >Send</v-btn
              >
            </div>
          </template>
        </div>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  data: () => ({
    item: '',
    template: null,
    templates: [],
    items: [],
    recipients: [],
    body_placeholders: [],
    header_url: null,
    sending: false,
  }),
  created() {
    this.loadTemplates()
  },
  methods: {
    loadTemplates() {
      this.$axios.get('/message-templates').then(({ data }) => {
        this.items = data.data.map((template, index) => {
          this.templates.push(template)
          return {
            text: `${template.name} (${template.language})`,
            value: index,
          }
        })
      })
    },
    selectTemplate() {
      this.template = this.templates[this.item]
      this.body_placeholders = []
      this.header_url = null
      this.formatTemplate()
    },
    send() {
      this.sending = true
      const payload = {
        recipients: this.recipients,
        header_url: this.header_url,
        header_type: this.template.header?.format || null,
        body_placeholders: this.body_placeholders,
        template_name: this.template.name,
        template_language: this.template.language,
      }
      this.$axios
        .post('/send-message-templates', payload)
        .then(({ data }) => {
          alert('Message(s) sucessfully sent!')
        })
        .catch((err) => {
          alert(err)
        })
        .finally(() => {
          this.sending = false
        })
    },
    formatTemplate() {
      this.template.header = null
      this.template.body = null
      this.template.footer = null
      this.template.buttons = null
      this.template.components.forEach((element) => {
        if (element.type === 'HEADER') this.template.header = element
        else if (element.type === 'BODY') {
          this.template.body = element.text
          this.template.body_placeholders = this.findPlaceholders(element.text)
        } else if (element.type === 'FOOTER')
          this.template.footer = element.text
        else if (element.type === 'BUTTONS')
          this.template.buttons = element.buttons
      })
    },
    findPlaceholders(text) {
      const regexp = /{{(.*?)}}/g
      const exec = text.matchAll(regexp)
      const matches = []

      for (const match of exec) {
        matches.push({ text: match[0], value: '' })
      }
      return matches
    },
  },
}
</script>

<style>
.pre-wrap {
  white-space: pre-wrap;
}
</style>