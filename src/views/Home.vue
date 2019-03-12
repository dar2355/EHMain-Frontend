<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <DynamicInput/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import DynamicInput from '@/components/DynamicInput.vue'; // @ is an alias to /src

export default Vue.extend({
  name: 'home',
  components: {
    DynamicInput,
  },
});
</script>
