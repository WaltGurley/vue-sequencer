<template>
  <h1>A Step Sequencer</h1>
  <Sequencer/>
</template>

<script>
import Sequencer from './components/Sequencer.vue'

export default {
  name: 'App',
  components: {
    Sequencer
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>
