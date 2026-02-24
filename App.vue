<template>
  <div id="app">
    <Quiz />
  </div>
</template>

<script>
import Quiz from './components/Quiz.vue'; 

export default {
  name: 'App',
  components: {
    Quiz,
  },
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
</style>