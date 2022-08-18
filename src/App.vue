<template>
  <div id="app">
    <CandidatesList/>
  </div>
</template>

<script>
import CandidatesList from './components/CandidatesList.vue'

export default {
  name: 'App',
  components: {
    CandidatesList
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  background-color: #eee;
  width: 560px;
  margin: auto;
  padding: 10px;
  border-radius: 10px;
  box-shadow: #9a9a9a 2px 2px 3px;
}
</style>
