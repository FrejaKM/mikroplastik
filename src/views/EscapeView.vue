<template>
    <Base postItUrl="/images/post-it-lime.png" title="Flugtruten"
        description="Mikroplast slipper ud i havet via forskellige ruter, du måske aldrig har tænkt over – gennem kloakker, afløb og med regnvand, der skyller ned i gadens riste."
        instructions="Træk og slip, så de de rette flugtruter passer til de rette mistænkte" levelId="flugtruten"
        :completed="true" @continue="handleContinue">
    <div>
        <img src="/images/escape_game.png" alt="" class="escape_game" />
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
        },
        handleContinue() {
            console.log('Fortsæt!');
        }
    }
};
</script>

<style scoped>
.escape_game {
    width: 100%;
    height: 100%;
    object-fit: cover;
}
</style>