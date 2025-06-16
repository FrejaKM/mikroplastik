<script setup>
import { RouterLink, RouterView } from 'vue-router'
import Board from './components/Board.vue'
</script>

<template>
  <router-view v-slot="{ Component }">
    <transition name="slide" mode="out-in">
      <Component :is="Component" />
    </transition>
  </router-view>
</template>

<style>
/* Slide transition 
.slide-enter-from,
.slide-leave-to {
  opacity: 0;
  transform: translateX(-100%);
}

.slide-enter-active,
.slide-leave-active {
  transition: 1s ease-out;
}

/* Ensure the transition container takes full space 
.slide-enter-active,
.slide-leave-active {
  will-change: transform, opacity;
}
*/
</style>