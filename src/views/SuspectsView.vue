<template>
  <Base postItUrl="/images/post-it-lime.png" title="De mistænkte"
    description="Mikroplast er efterhånden at finde alle vegne på vores planet.<br> <br>Et stykke plastik skal være mindre end 5 mm for at man kalder det mikroplastik.<br> <br>Lad os få taget billeder af mikroplasten, så vi kan efterlyse dem!"
    instructions="Træk de stykker plastik, der er mindre end 5 mm, ind til fotografering." levelId="suspects"
    :completed="true" @continue="handleContinue">
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
  width: 93%;
  height: 100%;
  object-fit: cover;
  scale: 0.95;
}
</style>