<template>
    <Base postItUrl="/images/post-it-green.png" title="Ofrene"
        description="Når mikroplast ender i havet, går det ud over dyrene. Heldigvis har forskerne fundet ud af, at mikroplasten ikke kommer videre i fødekæden, som man ellers frygtede. Det betyder, at vi ikke skal være bange for at få mikroplast i kroppen, når vi spiser fisk."
        instructions="Træk og slip mikroplasten hen på havdyrene og se, hvordan det reagerer på at spise det."
        levelId="ofrene" :completed="true" @continue="handleContinue">
    <div>
        <img src="/images/victims_game.png" alt="" class="victims_game" />
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

<style scoped>
.victims_game {
    display: block;
    margin: 0 auto;
    width: 100%;
    object-fit: cover;
}
</style>