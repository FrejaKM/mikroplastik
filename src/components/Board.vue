<template>
    <div class="detective-board">
        <!-- Title Section -->
        <div class="cork-board-container">
            <!-- Start Over Button -->
            <button class="start-over-button" :class="{ 'pulse-button': shouldPulseStartOver }" @click="startOver">
                
            </button>

            <div class="board-header">
                <h1 class="board-title">
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
                    backgroundImage: `url(${item.completionImage || '/images/ui/red-pin.png'})`,
                }" @animationend="onAnimationEnd(item.id)">
                </div>
            </div>

            <!-- Final completion image - appears when all levels are done -->
            <div v-if="allLevelsCompleted" class="final-completion-image" :class="{
                'fade-in': showFinalImage,
                'no-pointer-events': !showFinalImage
            }">
                <img src="/images/games+completion/final_file.png" alt="Case Completed" />

                <!-- Close button -->
                <button class="close-button" @click="closeFinalImage">

                </button>

                <!-- Stamp that appears on top of the file -->
                <div v-if="showStamp" class="stamp-image" :class="{ 'stamp-in': showStamp }">
                    <img src="/images/ui/stamp.png" alt="Case Stamped" />
                </div>
            </div>

        </div>
        <!-- Persistent logo in bottom left -->
        <div class="logo-container">
            <img src="/images/ui/logo_white.png" alt="Logo" class="logo" />
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue'
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// Completion tracking
const completedLevels = ref({})
const newlyCompletedLevels = ref(new Set())
const lastBoardVisit = ref(Date.now())
const showFinalImage = ref(false)
const showStamp = ref(false)
const shouldPulseStartOver = ref(false)

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

// Computed property to check if all levels are completed
const allLevelsCompleted = computed(() => {
    return levelOrder.every(levelId => completedLevels.value[levelId])
})

// Watch for when all levels become completed
watch(allLevelsCompleted, async (newVal) => {
    if (newVal) {
        // Small delay to let any animations finish
        await nextTick()
        setTimeout(() => {
            showFinalImage.value = true
            // Show stamp 1 second after the file appears
            setTimeout(() => {
                showStamp.value = true
                shouldPulseStartOver.value = true
            }, 2000)
        }, 2500)
    } else {
        showFinalImage.value = false
        showStamp.value = false
        shouldPulseStartOver.value = false
    }
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
        // Stop the pulsing
        shouldPulseStartOver.value = false

        // Clear completed levels
        completedLevels.value = {}
        localStorage.removeItem('completedLevels')

        // Clear individual level progress for all levels
        const levelProgressKeys = [
            'levelProgress_sporene',
            'levelProgress_escape',
            'levelProgress_ofrene',
            'levelProgress_suspects',
            'levelProgress_solution',
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
        image: '/images/post-it/post-it-green.png',
        completionImage: '/images/games+completion/victims.png',
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
        image: '/images/post-it/post-it-sand.png',
        completionImage: '/images/games+completion/kvittering.png',
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
        image: '/images/post-it/post-it-lime.png',
        completionImage: '/images/games+completion/poster.png',
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
        image: '/images/post-it/post-it-yellow.png',
        completionImage: '/images/games+completion/map.png',
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
        top: '50%',
        left: '65%',
        rotation: -3,
        route: '/solution',
        image: '/images/post-it/post-it-pink.png',
        completionImage: '/images/games+completion/habits.png',
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

// Close final completion image
const closeFinalImage = () => {
    showFinalImage.value = false
    showStamp.value = false
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
    background-color: #F1F4F0;
    overflow: hidden;
    padding: var(--padding-small);
    box-sizing: border-box;
    transform-origin: center;
}

.cork-board-container {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.9);
    width: 100%;
    height: 100%;
    background-image: url('/images/ui/cork-board.png');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    position: relative;
}

.start-over-button {
    position: absolute;
    background-color: transparent;
    left: var(--padding-small);
    background-size: 100%;
    background-position: center;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    background-repeat: no-repeat;
    color: rgb(0, 0, 0);
    padding: var(--padding-small);
    font-size: var(--font-medium);
    cursor: pointer;
    z-index: 1100;
    border: none;
    background-image: url('/images/ui/startover_btn.png');
    background-size: contain;
    background-repeat: no-repeat;
    width: 22vh;
    height: 10vh;
}

.start-over-button:hover {
    scale: 1.05;
}

.start-over-button.pulse-button {
    animation: pulse 2s ease-in-out infinite;
}

/* Header Section */
.board-header {
    position: absolute;
    top: 0.5vh;
    left: 50%;
    transform: translateX(-50%);
    z-index: 5;
    background-image: url('/images/ui/torn_paper.png');
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
    width: 50vw;
    height: 20vh;
    justify-content: center;
    align-items: center;
}

/* Post-it wrapper for positioning */
.post-it-wrapper {
    position: absolute;
    width: var(--post-it-size);
    height: var(--post-it-size);
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
    scale: 1.3;
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
    scale: 1.3 !important;
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
    width: var(--completion-image-size);
    height: var(--completion-image-size);
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

/* Final completion image styles */
.final-completion-image {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 1000;
    opacity: 0;
    transition: opacity 2s ease-in-out;
}

.final-completion-image:not(.fade-in) {
    transition: none;
}

.final-completion-image.fade-in {
    opacity: 1;
}

/* Critical fix: Disable pointer events when hidden */
.final-completion-image.no-pointer-events {
    pointer-events: none;
}

.final-completion-image img {
    width: 140vh;
    height: auto;
    object-fit: contain;
    filter: drop-shadow(0 0 20px rgba(0, 0, 0, 0.8));
}

/* Close button styles */
.close-button {
    background-image: url('/images/ui/x.png');
    background-size: contain;
    position: absolute;
    background-repeat: no-repeat;
    top: clamp(50px, 10vh, 110px);
    right: clamp(35px, 7vw, 110px);
    width: clamp(30px, 3vw, 50px);
    height: clamp(30px, 3vw, 50px);
    border: none;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1001;
    transition: all 0.2s ease;
}

.close-button:hover {
    transform: scale(1.1);
}

/* Stamp animation */
@keyframes stampIn {
    0% {
        transform: translate(-50%, -50%) scale(100);
        opacity: 0;
    }
    10% {
        opacity: 1;
    }
    50% {
        transform: translate(-50%, -50%) scale(0.8);
    }
    70% {
        transform: translate(-50%, -50%) scale(1.1);
    }
    85% {
        transform: translate(-50%, -50%) scale(0.95);
    }
    100% {
        transform: translate(-50%, -50%) scale(1);
        opacity: 1;
    }
}

.stamp-image {
    position: absolute;
    top: 54%;
    left: 76%;
    transform: translate(-50%, -50%);
    z-index: 1100;
    opacity: 0;
}

.stamp-image.stamp-in {
    animation: stampIn 1s ease-out forwards;
}

.stamp-image img {
    width: 50vh;
    height: auto;
    object-fit: contain;
    filter: drop-shadow(0 0 20px rgba(0, 0, 0, 0.0));
}

.logo-container {
    position: fixed;
    bottom: var(--padding-medium);
    right: var(--padding-medium);
    z-index: 100;
}

.logo {
    width: 30vh;
    height: auto;
}

@media (max-width: 1700px) {
    .completion-image {
        width: 40px !important;
        height: 40px !important;
    }
}
</style>