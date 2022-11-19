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
            @change="selectTemplate"
          ></v-select>

          <template v-if="template">
            <v-divider class="mb-4"></v-divider>
            <v-alert v-if="template.status !== 'APPROVED'" type="error"
              >Template unapproved and can't be used to send messages.</v-alert
            >
            <div v-if="template.header" class="my-3">
              <h5 class="text-h5">Header</h5>
              <p v-if="template.header.format == 'TEXT'">
                {{ template.header.text }}
              </p>
              <v-text-field
                v-else
                :label="`${template.header.format} URL`"
              ></v-text-field>
            </div>
            <div v-if="template.body" class="my-3">
              <h5 class="text-h5">Body</h5>
              <p class="pre-wrap">{{ template.body }}</p>
            </div>
            <div v-if="template.footer" class="my-3">
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
      this.formatTemplate()
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
      console.log(this.template)
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