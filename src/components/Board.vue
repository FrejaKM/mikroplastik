<template>
    <div class="detective-board">
        <div v-for="item in boardItems" :key="item.id" class="post-it" :class="item.category" :style="{
            top: item.top,
            left: item.left,
            transform: `rotate(${item.rotation}deg)`
        }" @click="navigateToCategory(item)">
            {{ item.title }}
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const boardItems = ref([
    {
        id: 'mistænkte',
        title: 'De mistænkte',
        category: 'suspects',
        top: '25%',
        left: '45%',
        rotation: -1,
        route: '/suspects'
    },
    {
        id: 'sporene',
        title: 'Sporene',
        category: 'clues',
        top: '60%',
        left: '75%',
        rotation: 2,
        route: '/clues'
    },
    {
        id: 'witnesses',
        title: 'Vidnerne',
        category: 'clues',
        top: '10%',
        left: '20%',
        rotation: 5,
        route: '/clues'
    },
    {
        id: 'evidence',
        title: 'Beviserne',
        category: 'suspects',
        top: '70%',
        left: '10%',
        rotation: -3,
        route: '/suspects'
    }
    // Add more items as needed
])

const navigateToCategory = (item) => {
    router.push(item.route)
}
</script>

<style scoped>
.detective-board {
    position: relative;
    width: 100vw;
    height: 100vh;

    ;
}

.post-it {
    position: absolute;
    padding: 15px;
    cursor: pointer;
    box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s ease;
}

.post-it:hover {
    transform: rotate(0deg) scale(1.05) !important;
    z-index: 10;
}

.suspects {
    background: #ffeb3b;
    /* yellow */
}

.clues {
    background: #4caf50;
    /* green */
}
</style>