<template>
    <div class="detective-board">
        <!-- Title Section -->
        <div class="cork-board-container">
            <!-- Start Over Button -->
            <button class="start-over-button" @click="startOver">
                Start efterfirskningen forfra
            </button>

            <div class="board-header">
                <h1 class="board-title">
                    <span class="title-text">Mysteriet om mikroplasten</span>
                </h1>
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
            <img src="/images/logo_white.png" alt="Logo" class="logo" />
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


// Start over function - now clears ALL progress and navigates to home
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

        // Navigate to home page
        router.push('/')
    }
}

const boardItems = ref([
    {
        id: 'ofrene',
        title: 'Ofrene',
        category: 'victims',
        top: '14%',
        left: '13%',
        rotation: -5,
        route: '/victims',
        image: '/images/post-it-green.png',
        completionImage: '/images/victims.png',
        completionPosition: {
            top: '37%',
            left: '18%',
            scale: 14,
            rotation: 0
        }
    },
    {
        id: 'sporene',
        title: 'Sporene',
        category: 'clues',
        top: '25%',
        left: '56%',
        rotation: 5,
        route: '/clues',
        image: '/images/post-it-sand.png',
        completionImage: '/images/kvittering.png',
        completionPosition: {
            top: '57%',
            left: '48%',
            scale: 15,
            rotation: 0
        }
    },
    {
        id: 'suspects',
        title: 'De mistænkte',
        category: 'suspects',
        top: '8%',
        left: '87%',
        rotation: 15,
        route: '/suspects',
        image: '/images/post-it-lime.png',
        completionImage: '/images/poster.png',
        completionPosition: {
            top: '39%',
            left: '85%',
            scale: 9,
            rotation: 0
        }
    },
    {
        id: 'flugtruten',
        title: 'Flugtruten',
        category: 'escape',
        top: '55%',
        left: '5%',
        rotation: 0,
        route: '/escape',
        image: '/images/post-it-yellow.png',
        completionImage: '/images/map.png',
        completionPosition: {
            top: '68%',
            left: '16%',
            scale: 18,
            rotation: 0
        }
    },
    {
        id: 'villain',
        title: 'Dommen',
        category: 'villain',
        top: '58%',
        left: '65%',
        rotation: -3,
        route: '/villain',
        image: '/images/post-it-pink.png',
        completionImage: '/images/shelf.png',
        completionPosition: {
            top: '55%',
            left: '8%',
            scale: 2,
            rotation: 0
        }
    },
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
    padding: 20px;
    box-sizing: border-box;
}

.cork-board-container {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.9);
    width: 100%;
    height: 100%;
    background-image: url('/images/cork-board.png');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    position: relative;
}

.start-over-button {
    font-family: 'ITP', sans-serif;
    font-weight: 300;
    position: absolute;
    top: 20px;
    left: 20px;
    background: #ffffff;
    color: rgb(0, 0, 0);
    border: 2px solid #5b5b5b;
    box-shadow: 3px 6px 5px rgba(0, 0, 0, 0.3);
    padding: 10px 20px;
    font-size: 20px;
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
    top: 9px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 5;
    background-image: url('/images/torn_paper.png');
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
    width: 50vw;
    height: 20vh;
    justify-content: center;
    align-items: center;
}

.board-title {
    position: absolute;
    top: 35%;
    left: 22%;
    margin: 0;
    z-index: 6;
}

.title-text {
    color: #000000;
    font-size: 2.2rem;
    line-height: 1.2;
    font-weight: 400;
    font-family: 'Coming Soon', cursive;
    text-align: center;
    font-weight: normal;
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
}

.post-it-text {
    color: #000000;
    font-weight: 400;
    text-align: center;
    font-family: 'Coming Soon', cursive;

    font-size: 1.4vw;
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
    transform-origin: center center;
}


.logo-container {
    position: fixed;
    bottom: 35px;
    right: 40px;
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