<template>
    <div class="detective-board">
        <!-- Title Section -->
        <div class="board-header">
            <h1 class="board-title">
                <span class="title-text">Mikroplast Mysteriet</span>
            </h1>
            <h2 class="board-subtitle">
                <span class="subtitle-text">En detektiv historie</span>
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
        top: '30%',
        left: '45%',
        rotation: -1,
        route: '/clues',
        image: '/images/post-it-yellow.png'
    },
    {
        id: 'suspects',
        title: 'De mistænkte',
        category: 'suspects',
        top: '55%',
        left: '65%',
        rotation: 2,
        route: '/suspects',
        image: '/images/post-it-green.png'
    },
    {
        id: 'ofrene',
        title: 'Ofrene',
        category: 'victims',
        top: '20%',
        left: '10%',
        rotation: 5,
        route: '/victims',
        image: '/images/post-it-sand.png'
    },
    {
        id: 'flugtruten',
        title: 'Flugtruten',
        category: 'escape',
        top: '70%',
        left: '10%',
        rotation: -3,
        route: '/escape',
        image: '/images/post-it-lime.png'
    },
    {
        id: 'skurk',
        title: 'En ny skurk',
        category: 'villain',
        top: '15%',
        left: '80%',
        rotation: 15,
        route: '/villain',
        image: '/images/post-it-white.png'
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
    /* cork board background */
}

/* Header Section */
.board-header {
    position: absolute;
    top: 30px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 5;
}

.board-title {
    position: relative;
    margin: 0px 0px 15px 0px;
    transform: rotate(-1deg);
}

.board-header {
    padding: 40px 80px;
    background-image: url('/images/torn-paper.png');
    background-size: 100% auto;
    background-repeat: no-repeat;
    background-position: center;

}

.title-text {
    color: #000000;
    font-size: 2.5rem;
    font-weight: bold;
    text-shadow: 1px 1px 3px rgba(255, 255, 255, 0.8);
    letter-spacing: 1px;
}

.board-subtitle {
    position: relative;
    margin: 0;
    padding: 12px 30px;
    transform: rotate(1deg);
}

.subtitle-text {
    color: #34495e;
    font-size: 1.2rem;
    font-weight: normal;
    text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
    font-style: italic;
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
    /* Remove background colors since we're using images */
}

.post-it:hover {
    transform: rotate(0deg) scale(1.05) !important;
    z-index: 10;
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

/* Responsive adjustments */
@media (max-width: 768px) {
    .title-text {
        font-size: 1.8rem;
    }

    .subtitle-text {
        font-size: 1rem;
    }

    .board-title,
    .board-subtitle {
        padding: 15px 25px;
    }

    .board-header {
        top: 20px;
    }
}
</style>