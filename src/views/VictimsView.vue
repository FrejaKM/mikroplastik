<template>
    <Base 
    postItUrl="/images/post-it/post-it-green.png" 
    instructionsUrl="/images/ui/instructions_victims.png"
    notebookUrl="/images/ui/victims_notebook.png"
    title="Flugtruten"
    levelId="flugtruten" 
    :completed="true"
    @continue="handleContinue">
    <div>
        <img src="/images/games+completion/victims_game.png" alt="" class="victims_game" />
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
    width: 120vh;
    object-fit: cover;
}
</style>