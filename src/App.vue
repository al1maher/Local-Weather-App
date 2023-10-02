<template>
  <div class="flex flex-col min-h-screen font-Roboto bg-gradient-to-b from-weather-primary to-indigo-500">
    <NavBar />
    <RouterView class="flex-1" v-slot="{ Component }">
      <Transition name="page">
        <component :is="Component" />
      </Transition>
    </RouterView>
  </div>
</template>

<script setup>
  import { RouterView } from 'vue-router';
  import NavBar from '../src/components/navbar.vue';
</script>

<style>
.page-enter-active,
.page-leave-active {
  transition: 600ms ease all;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
}
</style>