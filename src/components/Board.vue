<template>
    <div class="detective-board">
        <!-- Title Section -->
        <div class="cork-board-container">

            <div class="board-header">
                <h1 class="board-title">
                    <span class="title-text">Havet under pres</span>
                </h1>
                <h2 class="board-subtitle">
                    <span class="subtitle-text">Mysteriet om Mikroplasten</span>
                </h2>
            </div>

            <!-- Post-it notes -->
            <div v-for="item in boardItems" :key="item.id" class="post-it" :class="item.category" :style="{
                top: item.top,
                left: item.left,
                transform: `rotate(${item.rotation}deg)`,
                backgroundImage: `url(${item.image})`,
                backgroundSize: 'contain',
                backgroundRepeat: 'no-repeat',
                backgroundPosition: 'center'
            }" @click="navigateToCategory(item)">
                <span class="post-it-text">{{ item.title }}</span>
            </div>

        </div>
        <!-- Persistent logo in bottom left -->
        <div class="logo-container">
            <img src="/images/logo.png" alt="Logo" class="logo" />
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const boardItems = ref([
    {
        id: 'sporene',
        title: 'Sporene',
        category: 'clues',
        top: '32%',
        left: '55%',
        rotation: 5,
        route: '/clues',
        image: '/images/post-it-sand.png'
    },
    {
        id: 'escape',
        title: 'Flugtruten',
        category: 'escape',
        top: '55%',
        left: '70%',
        rotation: 0,
        route: '/escape',
        image: '/images/post-it-white.png'
    },
    {
        id: 'ofrene',
        title: 'Ofrene',
        category: 'victims',
        top: '20%',
        left: '13%',
        rotation: -5,
        route: '/victims',
        image: '/images/post-it-green.png'
    },
    {
        id: 'suspects',
        title: 'De mistænkte',
        category: 'escape',
        top: '59%',
        left: '5%',
        rotation: -3,
        route: '/suspects',
        image: '/images/post-it-yellow.png'
    },
    {
        id: 'skurk',
        title: 'En ny skurk',
        category: 'villain',
        top: '15%',
        left: '88%',
        rotation: 15,
        route: '/villain',
        image: '/images/post-it-lime.png'
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
    background: #F0F4F0;
    overflow: hidden;
    padding: 30px;
    box-sizing: border-box;
}

.cork-board-container {
    width: 100%;
    height: 100%;
    background-image: url('/images/cork-board.png');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    position: relative;
}

/* Header Section */
.board-header {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 5;
    background-image: url('/images/Group 20.png');
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
    width: 50vw;
    /* Set a specific width instead of 100% */
    height: 23vh;
    /* Set a specific height */
    justify-content: left;
    align-items: left;
    /* Remove the scale property */
}

/* Title positioning on top of the PNG */
.board-title {
    position: absolute;
    top: 30%;
    /* Adjust this to position on the paper */
    left: 50%;
    transform: translateX(-50%);
    margin: 0;
    z-index: 6;
}

.board-subtitle {
    position: absolute;
    top: 55%;
    /* Adjust this to position below the title */
    left: 50%;
    transform: translateX(-50%);
    margin: 0;
    z-index: 6;
}

.title-text {
    color: #000000;
    font-size: 1.2rem;
    line-height: 1.2;
    margin: 0;
    text-align: center;
    font-weight: normal;
}

.subtitle-text {
    color: #000000;
    font-size: 1.8rem;
    font-weight: bold;
    margin: 0;
    text-align: center;
    font-family: 'Arial', sans-serif;
}

.post-it {
    position: absolute;
    width: 120px;
    height: 120px;
    cursor: pointer;
    transition: transform 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 3;
    /* Lower than header but still interactive */
}

.post-it:hover {
    transform: rotate(0deg) !important;
    z-index: 10;
    /* Bring to front on hover */
}

.post-it-text {
    color: #333;
    font-weight: bold;
    text-align: center;
    font-size: 14px;
    text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
    z-index: 2;
    padding: 5px;
}

.logo-container {
    position: fixed;
    bottom: 6px;
    right: 8px;
    z-index: 1000;
}

.logo {
    width: 15vh;
    height: auto;
}

/* Responsive adjustments */
@media (max-width: 768px) {
    .board-header {
        width: 300px;
        height: 150px;
    }

    .title-text {
        font-size: 1rem;
    }

    .subtitle-text {
        font-size: 1.4rem;
    }
}
</style>