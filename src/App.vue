<script setup>
import { RouterLink, RouterView } from 'vue-router'
import Board from './components/Board.vue'
</script>

<template>
  <div class="wrapper">

    <div class="cork-board-container">
      <router-view v-slot="{ Component }">
        <transition name="grow-in" mode="out-in">
          <Component :is="Component" />
        </transition>
      </router-view>
    </div>
  </div>
</template>

<style>
.wrapper {
  position: absolute;
  width: 100vw;
  height: 100vh;
  background: #F0F4F0;
  overflow: hidden;
  padding: 20px;
  box-sizing: border-box;
}

.cork-board-container {
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.9);
  width: 100%;
  height: 100%;
  background-image: url('/images/cork-board.png');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  position: relative;
}

/* Grow-in transition */
.grow-in-enter-from,
.grow-in-leave-to {
  opacity: 0;
  transform: scale(0.3);
}

.grow-in-enter-active,
.grow-in-leave-active {
  transition: 1s ease-out;
}

/* Ensure the transition container takes full space */
.grow-in-enter-active,
.grow-in-leave-active {
  will-change: transform, opacity;
}
</style>