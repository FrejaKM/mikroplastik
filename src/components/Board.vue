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
                <div class="post-it" :class="[
                    item.category,
                    {
                        completed: completedLevels[item.id],
                        available: isLevelAvailable(item.id),
                        locked: !isLevelAvailable(item.id) && !completedLevels[item.id]
                    }
                ]" :style="{
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
                <div v-if="completedLevels[item.id]" :data-completion-id="item.id" :class="[
                    'completion-image',
                    { 'bounce-in': newlyCompletedLevels.has(item.id) }
                ]" :style="{
                    position: 'absolute',
                    top: item.completionPosition?.top || item.top,
                    left: item.completionPosition?.left || item.left,
                    '--final-scale': item.completionPosition?.scale || 1,
                    '--final-rotation': (item.completionPosition?.rotation || 0) + 'deg',
                    transform: getCompletionTransform(item),
                    backgroundImage: `url(${item.completionImage || '/images/red-pin.png'})`,
                }" @animationend="onAnimationEnd(item.id)">
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
import { ref, onMounted, computed, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// Completion tracking
const completedLevels = ref({})
const newlyCompletedLevels = ref(new Set())
const lastBoardVisit = ref(Date.now())

// Define the order of levels
const levelOrder = ['sporene', 'suspects', 'flugtruten', 'ofrene', 'solution']

// Computed property to determine which level should be available next
const nextAvailableLevel = computed(() => {
    for (let i = 0; i < levelOrder.length; i++) {
        if (!completedLevels.value[levelOrder[i]]) {
            return levelOrder[i]
        }
    }
    return null // All levels completed
})

// Function to check if a level is available
const isLevelAvailable = (levelId) => {
    // First level (suspects) is always available
    if (levelId === 'sporene') return true

    // Check if this is the next level in sequence
    return levelId === nextAvailableLevel.value
}

// Load completion state from localStorage
const loadCompletedLevels = () => {
    const saved = localStorage.getItem('completedLevels')
    if (saved) {
        completedLevels.value = JSON.parse(saved)
    }
}

// Check for newly completed levels since last board visit
const checkForNewCompletions = () => {
    const lastVisit = localStorage.getItem('lastBoardVisit')
    const lastVisitTime = lastVisit ? parseInt(lastVisit) : 0

    // Check each level's completion timestamp
    const saved = localStorage.getItem('completedLevels')
    if (saved) {
        const completedData = JSON.parse(saved)

        // Find levels completed since last visit
        Object.keys(completedData).forEach(levelId => {
            const completionTime = localStorage.getItem(`completion_time_${levelId}`)
            if (completionTime && parseInt(completionTime) > lastVisitTime) {
                newlyCompletedLevels.value.add(levelId)
            }
        })
    }

    // Update last visit time
    localStorage.setItem('lastBoardVisit', Date.now().toString())
}

// Save completion state to localStorage
const saveCompletedLevels = () => {
    localStorage.setItem('completedLevels', JSON.stringify(completedLevels.value))
}

// Mark level as completed
const markLevelCompleted = (levelId) => {
    completedLevels.value[levelId] = true
    // Store completion timestamp
    localStorage.setItem(`completion_time_${levelId}`, Date.now().toString())
    saveCompletedLevels()
}

// Handle animation end and set final transform
const onAnimationEnd = (levelId) => {
    newlyCompletedLevels.value.delete(levelId)
    // Force update to apply final transform
    const element = document.querySelector(`[data-completion-id="${levelId}"]`)
    if (element) {
        const item = boardItems.value.find(i => i.id === levelId)
        if (item) {
            element.style.transform = `scale(${item.completionPosition?.scale || 1}) rotate(${item.completionPosition?.rotation || 0}deg)`
        }
    }
}

// Get the final transform for completion images
const getCompletionTransform = (item) => {
    if (newlyCompletedLevels.value.has(item.id)) {
        return 'scale(0)' // Start animation from scale 0
    }
    return `scale(${item.completionPosition?.scale || 1}) rotate(${item.completionPosition?.rotation || 0}deg)`
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
        top: '53%',
        left: '10%',
        rotation: 0,
        route: '/escape',
        image: '/images/post-it-yellow.png',
        completionImage: '/images/map.png',
        completionPosition: {
            top: '73%',
            left: '16%',
            scale: 12,
            rotation: 0
        }
    },
    {
        id: 'solution',
        title: 'Opklaringen',
        category: 'solution',
        top: '52%',
        left: '65%',
        rotation: -3,
        route: '/solution',
        image: '/images/post-it-pink.png',
        completionImage: '/images/habits.png',
        completionPosition: {
            top: '77%',
            left: '75%',
            scale: 10,
            rotation: 0
        }
    },
])

const navigateToCategory = (item) => {
    // Only allow navigation if the level is available or already completed
    if (isLevelAvailable(item.id) || completedLevels.value[item.id]) {
        router.push(item.route)
    } else {
        // Optional: Show a message that this level is not yet available
        console.log(`Level ${item.title} is not yet available`)
    }
}

// Check for completion when returning from a level
onMounted(() => {
    loadCompletedLevels()
    checkForNewCompletions()

    // Listen for completion events (for when levels are completed while on board)
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
    background-color: transparent;
    overflow: hidden;
    padding: 20px;
    box-sizing: border-box;
    transform-origin: center;
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
    position: absolute;
    background-image: url('/images/button_short.png');
    top: 20px;
    background-color: transparent;
    left: 15px;
    width: 230px;
    background-size: 100%;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    background-repeat: no-repeat;
    color: rgb(0, 0, 0);
    padding: 20px;
    font-size: 20px;
    cursor: pointer;
    z-index: 1000;
    border: none;
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
    transition: transform 0.2s ease, opacity 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
    position: relative;
}

.post-it:hover {
    transform: rotate(0deg) scale(1.02) !important;
    z-index: 10;
    filter: brightness(1.05);
}

.post-it.completed {
    z-index: 10;
}

/* Available level - pulsing animation (but not if completed) */
.post-it.available:not(.completed) {
    animation: pulse 2s ease-in-out infinite;
    cursor: pointer;
}

/* Locked level - reduced opacity and no cursor */
.post-it.locked {
    filter: brightness(0);
    cursor: not-allowed;
}

.post-it.locked:hover {
    transform: none !important;
    scale: 1 !important;
}

.post-it-text {
    color: #000000;
    font-weight: 400;
    text-align: center;
    font-family: 'Coming Soon', cursive;
    font-size: 1.6vw;
    text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
    z-index: 2;
    padding: 5px;
}

/* Pulsing animation for available levels */
@keyframes pulse {
    0% {
        transform: scale(1);
    }

    50% {
        transform: scale(1.1);
    }

    100% {
        transform: scale(1);
    }
}

/* Bouncy animation for newly completed levels */
@keyframes bounceIn {
    0% {
        transform: scale(0) rotate(var(--final-rotation, 0deg));
        opacity: 0;
    }

    50% {
        transform: scale(calc(var(--final-scale, 1) * 1.2)) rotate(var(--final-rotation, 0deg));
        opacity: 1;
    }

    70% {
        transform: scale(calc(var(--final-scale, 1) * 0.9)) rotate(var(--final-rotation, 0deg));
    }

    85% {
        transform: scale(calc(var(--final-scale, 1) * 1.05)) rotate(var(--final-rotation, 0deg));
    }

    100% {
        transform: scale(var(--final-scale, 1)) rotate(var(--final-rotation, 0deg));
        opacity: 1;
    }
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

/* Bounce animation for newly completed levels */
.completion-image.bounce-in {
    animation: bounceIn 0.8s ease-out forwards;
    animation-fill-mode: forwards;
}

/* After animation, apply the final transform */
.completion-image:not(.bounce-in) {
    transition: transform 0.2s ease;
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