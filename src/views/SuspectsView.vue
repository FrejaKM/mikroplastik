<template>
  <Base postItUrl="/images/post-it-yellow.png" title="Ofrene"
    description="Når mikroplasten slipper ud i havet, er det oftest havets dyr, det går ud over. Selvom mikroplast helt sikkert ikke hører til i havet, så har det vist sig, at det heldigvis ikke påvirker fødekæden, som man ellers havde frygtet. Vi mennesker skal altså ikke være bange for at komme til at få mikroplast med indenbords, når vi spiser fisk."
    instructions="Test Instruktioner" :completed="true" @continue="handleContinue">
  <div style="background: red; height: 100px; width: 100%; text-align: center; color: white;">
    TEST SPILINDHOLD
  </div>
  </Base>
</template>

<script>
import Base from '../components/Base.vue';

export default {
  name: 'YourView',
  components: {
    Base
  },
  methods: {
    handleContinue() {
      console.log('Fortsæt!');
    }
  }
};
</script>