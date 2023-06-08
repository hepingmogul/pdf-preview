<template>
  <div id="app">
    <pdf-preview :url="url"></pdf-preview>
  </div>
</template>

<script>
import pdfPreview from './components/PdfLaypage.vue'

export default {
  name: 'app',
  components: {
    pdfPreview
  },
  data () {
    return {
      url: '/static/EUNOMIA2022年欧洲PET市场报告英文版.pdf'
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 700px;
}
</style>
