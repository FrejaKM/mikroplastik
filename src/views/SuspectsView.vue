<template>
  <Base postItUrl="/images/post-it-lime.png" title="De mistænkte" description="
Mikroplast er efterhånden at finde alle vegne på vores planet, men det er meget svært at opdage.  
Et stykke plastik skal nemlig være mindre end 5 millimeter, for at man kalder det for mikroplastik. "
    instructions="Som en del af efterforskningen skal der tages billeder af de mistænkte i sagen. Træk de stykker plastik ind, som har mikroplast-størrelse (mindre end 5 millimeter)."
    levelId="suspects" :completed="true" @continue="handleContinue">
  <div>
    <img src="/images/suspects_game.png" alt="" class="suspects_game" />
  </div>
  </Base>
</template>

<script>
import Base from '../components/Base.vue';

export default {
  name: 'SuspectsView',
  components: {
    Base
  },
  mounted() {
    // Automatically mark as completed when mounted
    this.markAsCompleted();
  },
  methods: {
    markAsCompleted() {
      const saved = localStorage.getItem('completedLevels');
      const completedLevels = saved ? JSON.parse(saved) : {};
      completedLevels['suspects'] = true;
      localStorage.setItem('completedLevels', JSON.stringify(completedLevels));
    },
    handleContinue() {
      console.log('Fortsæt!');
    }
  }
};
</script>

<style scoped>
.suspects_game {
  width: 85%;
  height: 100%;
  object-fit: cover;
  scale: 0.95;
  display: block;
  justify-self: center;
}
</style>