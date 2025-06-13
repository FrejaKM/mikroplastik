<template>
    <Base postItUrl="/images/post-it-green.png" title="Ofrene"
        description="Når mikroplasten slipper ud i havet, er det oftest havets dyr, det går ud over. Selvom mikroplast helt sikkert ikke hører til i havet, så har det vist sig, at det heldigvis ikke påvirker fødekæden, som man ellers havde frygtet. Vi mennesker skal altså ikke være bange for at komme til at få mikroplast med indenbords, når vi spiser fisk."
        instructions="Test Instruktioner" levelId="ofrene" :completed="true" @continue="handleContinue">
    <div style="background: red; height: 100px; width: 100%; text-align: center; color: white;">
        TEST SPILINDHOLD
    </div>
    </Base>
</template>

<script>
import Base from '../components/Base.vue';

export default {
    name: 'VictimsView',
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
            completedLevels['ofrene'] = true; // Fixed: changed from 'ofre' to 'ofrene'
            localStorage.setItem('completedLevels', JSON.stringify(completedLevels));
        },
        handleContinue() {
            console.log('Fortsæt!');
        }
    }
};
</script>