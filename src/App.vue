<template>
  <div id="app" class="container">
    <YachtSee />
  </div>
</template>

<script>
import YachtSee from './components/YachtSee';

export default {
  name: 'App',
  components: {
    YachtSee,
  },
};
</script>

<style>
#app {
  margin: 0 auto;
  padding-top: 5rem;
  padding-bottom: 1rem;
  max-width: 600px;
}
</style>
