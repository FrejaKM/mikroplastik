<template>
  <Base 
    postItUrl="/images/post-it/post-it-lime.png" 
    instructionsUrl="/images/ui/instructions_suspects.png"
    notebookUrl="/images/ui/suspects_notebook.png"
    title="De mistænkte" 
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
  width: 100vh;
  object-fit: cover;
}
</style>