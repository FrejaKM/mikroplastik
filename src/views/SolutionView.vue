<template>
    <Base postItUrl="/images/post-it-pink.png" title="Opklaringen"
        description="Sagen er næsten opklaret. Du har identificeret og fundet mikroplasten, fulgt dens vej ud mod havet og set, hvem det går ud over. <br> <br> Men én vigtig brik mangler: Hvordan forhindrer vi, at sagen gentager sig?"
        instructions="Hvordan kan vi stoppe mikroplast fra at ende i havet?" levelId="solution" :completed="true"
        @continue="handleContinue">
    <div style="background: red; height: 100px; width: 100%; text-align: center; color: white;">
        TEST SPILINDHOLD
    </div>
    </Base>
</template>

<script>
import Base from '../components/Base.vue';

export default {
    name: 'SolutionView',
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
            completedLevels['solution'] = true; // Fixed: changed from 'skurk' to 'villain'
            localStorage.setItem('completedLevels', JSON.stringify(completedLevels));
        },
        handleContinue() {
            console.log('Fortsæt!');
        }
    }
};
</script>