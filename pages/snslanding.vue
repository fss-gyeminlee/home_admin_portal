<template>
  <v-row>
    <v-col class="text-center" cols="4">
      <div>
        <ckeditor
          id="editor"
          ref="editor"
          v-model="html"
          :config="editorConfig"
        ></ckeditor>
      </div>
    </v-col>
    <v-col class="text-center" cols="8">
      <iframe
        ref="iframe"
        src="http://localhost:8181/content/page/sns-landing"
        width="100%"
        height="1200px"
      >
      </iframe>
    </v-col>
    <v-btn depressed color="primary" @click="buttonClick"> 저장 </v-btn>
  </v-row>
</template>

<script>
import axios from 'axios'

let CKEditor

if (process.client) {
  CKEditor = require('ckeditor4-vue')
} else {
  CKEditor = { component: { template: '<div></div>' } }
}

export default {
  name: 'InspirePage',
  components: {
    ckeditor: CKEditor.component,
  },
  data() {
    return {
      html: '',
      editorData: '<p>Content of the editor.</p>',
      editorConfig: {
        // The configuration of the editor.
        id: 'gmlee',
        height: '1050px',
        width: '100%',
        startupMode: 'source',
        toolbarGroups: [{ name: 'tools', groups: ['tools'] }],
      },
    }
  },
  mounted: function () {
    axios
      .get('http://localhost:3000/api/landing/sns-landing')
      .then((response) => {
        let temp1 = response.data.html
        temp1 = temp1.slice(1, temp1.length)
        temp1 = temp1.slice(0, temp1.length - 1)
        const unEscapeHTML = temp1
          .replace(/&amp;/g, '&')
          .replace(/&gt;/g, '>')
          .replace(/&lt;/g, '<')
          .replace(/&quot;/g, '"')
          .replace(/\\n/g, '\n')
          .replace(/\\t/g, '\t')
        this.html = unEscapeHTML
      })
    document.addEventListener(
      'keydown',
      function (e) {
        if (
          e.key === 's' &&
          (navigator.platform.match('Mac') ? e.metaKey : e.ctrlKey)
        ) {
          e.preventDefault()

          const arr = Object.keys(CKEDITOR.instances)

          const escapeHTML = `'${CKEDITOR.instances[arr[0]].getData()}'`
            .replace(/&/g, '&amp;')
            .replace(/>/g, '&gt;')
            .replace(/</g, '&lt;')
            .replace(/"/g, '&quot;')

          const data = {
            html: escapeHTML,
          }
          axios
            .put('http://localhost:3000/api/landing/sns-landing', data)
            .then(() => {
              const src = this.$refs.iframe.src
              this.$refs.iframe.src = src
            })
        }
      }.bind(this),
      false
    )
  },
  methods: {
    buttonClick() {
      // var ckData = CKEditor.instances.editor1.getData()

      const arr = Object.keys(CKEDITOR.instances)

      const escapeHTML = `'${CKEDITOR.instances[arr[0]].getData()}'`
        .replace(/&/g, '&amp;')
        .replace(/>/g, '&gt;')
        .replace(/</g, '&lt;')
        .replace(/"/g, '&quot;')

      const data = {
        html: escapeHTML,
      }
      axios
        .put('http://localhost:3000/api/landing/sns-landing', data)
        .then(() => {
          const src = this.$refs.iframe.src
          alert('update success')
          this.$refs.iframe.src = src
        })
    },
    onKeyDown(e) {
      if ((e.which === '115' || e.which === '83') && (e.ctrlKey || e.metaKey)) {
        e.preventDefault()
        return false
      }
      return true
    },
  },
}
</script>
