<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <novoEdior></novoEdior>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import novoEdior from '@/components/editorJs/data-Add.vue'

export default {
  name: 'HomeView',
  components: {
    HelloWorld,
    novoEdior
  }
}
</script>
