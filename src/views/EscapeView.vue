<template>
    <Base 
    postItUrl="/images/post-it/post-it-yellow.png" 
    notebookUrl="/images/ui/escape_notebook.png"
    title="Flugtruten"
    instructions="Tegn pile, så flugtruterne passer." 
    levelId="flugtruten" 
    :completed="true"
    @continue="handleContinue">
    <div>
        <img src="/images/games+completion/escape_game.png" alt="" class="escape_game" />
    </div>
    </Base>
</template>

<script>
import Base from '../components/Base.vue';

export default {
    name: 'EscapeView',
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
            completedLevels['flugtruten'] = true;
            localStorage.setItem('completedLevels', JSON.stringify(completedLevels));

            // Store completion timestamp for animation detection
            localStorage.setItem(`completion_time_flugtruten`, Date.now().toString());
        },
        handleContinue() {
            console.log('Fortsæt!');
        }
    }
};
</script>

<style scoped>
.escape_game {
    display: block;
    margin: 0 auto;
    width: 100%;
    object-fit: cover;
}
</style>