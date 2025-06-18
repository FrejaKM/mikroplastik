<template>
  <Base 
    postItUrl="/images/post-it/post-it-lime.png" 
    notebookUrl="/images/ui/suspects_notebook.png"
    title="De mistænkte" 
    instructions="Som en del af efterforskningen skal der tages billeder af de mistænkte i sagen. Træk de stykker plastik ind, som har mikroplast-størrelse (mindre end 5 millimeter)."
    levelId="suspects" 
    :completed="true" 
    @continue="handleContinue">

  <div>
    <img src="/images/games+completion/suspects_game.png" alt="" class="suspects_game" />
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
  display: block;
  margin: 0 auto;
  width: 60vw;
  object-fit: cover;
}
</style>