
<style>
@import './assets/main.css'; /* Import Tailwind */
</style>
<template>
  <div id="app">
    <section class="container mx-auto">
      <UserList />
    </section>
   
  </div>
</template>

<script>
import UserList from './components/UserList.vue';

export default {
  components: {
    UserList,
  },
};
</script>


