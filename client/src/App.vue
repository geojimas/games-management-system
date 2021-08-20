<template>
  <Header />
  <Content />
  <Footer />
</template>

<script>
import Content from './components/Content.vue'
import Footer from './components/Footer'
import Header from './components/Header'

export default {
  name: 'App',
  components: {
    Content,
    Header,
    Footer,
  },
}
</script>
