<template>
    <div class="detective-board">
        <!-- Title Section -->
        <div class="cork-board-container">
            <!-- Start Over Button -->
            <button class="start-over-button" @click="startOver">
                Start forfra
            </button>

            <div class="board-header">
                <h1 class="board-title">
                    <span class="title-text">Havet under pres</span>
                </h1>
                <h2 class="board-subtitle">
                    <span class="subtitle-text">Mysteriet om Mikroplasten</span>
                </h2>
            </div>

            <!-- Post-it notes -->
            <div v-for="item in boardItems" :key="item.id" class="post-it-wrapper" :style="{
                top: item.top,
                left: item.left,
                transform: `rotate(${item.rotation}deg)`,
            }">
                <!-- Post-it note -->
                <div class="post-it" :class="[item.category, { completed: completedLevels[item.id] }]" :style="{
                    backgroundImage: `url(${item.image})`,
                    backgroundSize: 'contain',
                    backgroundRepeat: 'no-repeat',
                    backgroundPosition: 'center'
                }" @click="navigateToCategory(item)">
                    <span class="post-it-text">{{ item.title }}</span>
                </div>
            </div>

            <!-- Completion images positioned independently -->
            <div v-for="item in boardItems" :key="`completion-${item.id}`">
                <div v-if="completedLevels[item.id]" class="completion-image" :style="{
                    position: 'absolute',
                    top: item.completionPosition?.top || item.top,
                    left: item.completionPosition?.left || item.left,
                    transform: `scale(${item.completionPosition?.scale || 1}) rotate(${item.completionPosition?.rotation || 0}deg)`,
                    backgroundImage: `url(${item.completionImage || '/images/red-pin.png'})`,
                }">
                </div>
            </div>

        </div>
        <!-- Persistent logo in bottom left -->
        <div class="logo-container">
            <img src="/images/logo_hvid.png" alt="Logo" class="logo" />
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

// Completion tracking
const completedLevels = ref({})

// Load completion state from localStorage
const loadCompletedLevels = () => {
    const saved = localStorage.getItem('completedLevels')
    if (saved) {
        completedLevels.value = JSON.parse(saved)
    }
}

// Save completion state to localStorage
const saveCompletedLevels = () => {
    localStorage.setItem('completedLevels', JSON.stringify(completedLevels.value))
}

// Mark level as completed
const markLevelCompleted = (levelId) => {
    completedLevels.value[levelId] = true
    saveCompletedLevels()
}

// Start over function - now clears ALL progress
const startOver = () => {
    if (confirm('Er du sikker på, at du vil starte forfra? Dette vil nulstille alle fremskridt.')) {
        // Clear completed levels
        completedLevels.value = {}
        localStorage.removeItem('completedLevels')

        // Clear individual level progress for all levels
        const levelProgressKeys = [
            'levelProgress_sporene',
            'levelProgress_escape',
            'levelProgress_ofrene',
            'levelProgress_suspects',
            'levelProgress_skurk'
            // Add more level progress keys as you create more levels
        ]

        levelProgressKeys.forEach(key => {
            localStorage.removeItem(key)
        })

        console.log('All progress cleared')
    }
}

const boardItems = ref([
    {
        id: 'sporene',
        title: 'Sporene',
        category: 'clues',
        top: '32%',
        left: '55%',
        rotation: 5,
        route: '/clues',
        image: '/images/post-it-sand.png',
        completionImage: '/images/kvittering.png',
        completionPosition: {
            top: '62%',
            left: '48%',
            scale: 13,
            rotation: 0
        }
    },
    {
        id: 'escape',
        title: 'Flugtruten',
        category: 'escape',
        top: '55%',
        left: '70%',
        rotation: 0,
        route: '/escape',
        image: '/images/post-it-white.png',
        completionImage: '/images/red-pin.png',
        completionPosition: {
            top: '51%',
            left: '73%',
            scale: 1.0,
            rotation: -10
        }
    },
    {
        id: 'ofrene',
        title: 'Ofrene',
        category: 'victims',
        top: '20%',
        left: '13%',
        rotation: -5,
        route: '/victims',
        image: '/images/post-it-green.png',
        completionImage: '/images/red-pin.png',
        completionPosition: {
            top: '16%',
            left: '16%',
            scale: 1.5,
            rotation: 25
        }
    },
    {
        id: 'suspects',
        title: 'De mistænkte',
        category: 'suspects',
        top: '59%',
        left: '5%',
        rotation: -3,
        route: '/suspects',
        image: '/images/post-it-yellow.png',
        completionImage: '/images/red-pin.png',
        completionPosition: {
            top: '55%',
            left: '8%',
            scale: 1.3,
            rotation: 0
        }
    },
    {
        id: 'skurk',
        title: 'En ny skurk',
        category: 'villain',
        top: '15%',
        left: '88%',
        rotation: 15,
        route: '/villain',
        image: '/images/post-it-lime.png',
        completionImage: '/images/red-pin.png',
        completionPosition: {
            top: '11%',
            left: '91%',
            scale: 0.8,
            rotation: -20
        }
    }
])

const navigateToCategory = (item) => {
    router.push(item.route)
}

// Check for completion when returning from a level
onMounted(() => {
    loadCompletedLevels()

    // Listen for completion events
    window.addEventListener('levelCompleted', (event) => {
        markLevelCompleted(event.detail.levelId)
    })
})
</script>

<style scoped>
.detective-board {
    position: absolute;
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

.start-over-button {
    position: absolute;
    top: 25px;
    left: 20px;
    background: #ffffff;
    color: rgb(0, 0, 0);
    border: 2px solid #000;
    box-shadow: 3px 6px 5px rgba(0, 0, 0, 0.5);
    padding: 10px 20px;
    font-size: 1vw;
    font-weight: bold;
    cursor: pointer;
    z-index: 1000;
    transition: background-color 0.2s ease;
}

.start-over-button:hover {
    scale: 1.05;
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
    height: 23vh;
    justify-content: left;
    align-items: left;
}

/* Title positioning on top of the PNG */
.board-title {
    position: absolute;
    top: 30%;
    left: 50%;
    transform: translateX(-50%);
    margin: 0;
    z-index: 6;
}

.board-subtitle {
    position: absolute;
    top: 55%;
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

/* Post-it wrapper for positioning */
.post-it-wrapper {
    position: absolute;
    width: 10vw;
    height: 10vw;
    z-index: 10;
}

.post-it {
    width: 100%;
    height: 100%;
    cursor: pointer;
    transition: transform 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
    position: relative;
}

.post-it:hover {
    transform: rotate(0deg) scale(1.02) !important;
    z-index: 10;
}

.post-it.completed {
    z-index: 10;
    /* Slightly dimmed when completed */
}

.post-it-text {
    color: #333;
    font-weight: bold;
    text-align: center;
    font-size: 1.3vw;
    text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
    z-index: 2;
    padding: 5px;
}

/* Completion image (red pin/checkmark) */
.completion-image {
    width: 40px;
    height: 40px;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    z-index: 9;
    animation: pinDrop 0.5s ease-out;
    transform-origin: center center;
}

@keyframes pinDrop {
    0% {
        transform: translateY(-20px) scale(0.5);
        opacity: 0;
    }

    50% {
        transform: translateY(5px) scale(1.1);
        opacity: 1;
    }

    100% {
        transform: translateY(0) scale(1);
        opacity: 1;
    }
}

.logo-container {
    position: fixed;
    bottom: 50px;
    right: 57px;
    z-index: 1000;
}

.logo {
    width: 15vw;
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