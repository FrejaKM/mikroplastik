<template>
    <div class="clues-game">
        <!-- Post-it note with title -->
        <div class="post-it-note">
            <h2>Sporene</h2>
        </div>

        <!-- Main content area -->
        <div class="game-container">
            <!-- Left side - Notebook with clue -->
            <div class="notebook-container">
                <div class="notebook-page">
                    <p>Mikroplast er efterhånden at finde alle vegne på vores planet. Men hvornår kalder man egentlig
                        noget for mikroplast?</p>
                </div>
            </div>

            <!-- Right side - Mugshot and size options -->
            <div class="mugshot-container">
                <!-- Mugshot background (Drop zone) -->
                <div class="mugshot-background" :class="{ 'drop-zone-active': isOverDropZone }"
                    @mouseenter="onDropZoneEnter" @mouseleave="onDropZoneLeave">
                    <img src="/images/mugshot.png" alt="Mugshot background" class="mugshot-image" />

                    <!-- Success message -->
                    <div v-if="gameCompleted" class="success-overlay">
                        <div class="success-text">Korrekt!</div>
                    </div>
                </div>

                <!-- Microplastic size options -->
                <div class="size-options">
                    <div v-for="item in items" :key="item.id" :data-item-id="item.id" :class="[
                        'plastic-piece',
                        {
                            'dragging': draggedItem?.id === item.id,
                            'incorrect': incorrectItems.has(item.id),
                            'collected': item.collected
                        }
                    ]" :style="{ opacity: item.opacity }" @mousedown="startDrag(item, $event)">
                        <img :src="item.image" :alt="item.size" />
                    </div>
                </div>
            </div>
        </div>

        <!-- Drag ghost element -->
        <div v-if="draggedItem" class="drag-ghost" :style="{
            left: mousePosition.x + 'px',
            top: mousePosition.y + 'px'
        }">
            <img :src="draggedItem.image" :alt="draggedItem.size" />
        </div>

        <!-- Bottom decorative elements -->
        <div class="bottom-elements">
            <div class="green-arrow"></div>
            <div class="blue-triangle"></div>
        </div>

        <!-- Continue button -->
        <button v-if="gameCompleted" class="continue-btn" @click="continueToNext">
            Fortsæt →
        </button>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

// Game state
const draggedItem = ref(null)
const mousePosition = ref({ x: 0, y: 0 })
const isOverDropZone = ref(false)
const gameCompleted = ref(false)
const incorrectItems = ref(new Set())

// Microplastic size options with PNG images
const items = ref([
    {
        id: 1,
        size: '5000μm',
        image: '/images/microplastic/plastic-5000.png',
        isCorrect: false,
        opacity: 1,
        collected: false
    },
    {
        id: 2,
        size: '100μm',
        image: '/images/microplastic/plastic-100.png',
        isCorrect: true,
        opacity: 1,
        collected: false
    },
    {
        id: 3,
        size: '10μm',
        image: '/images/microplastic/plastic-10.png',
        isCorrect: false,
        opacity: 1,
        collected: false
    },
    {
        id: 4,
        size: '1μm',
        image: '/images/microplastic/plastic-1.png',
        isCorrect: false,
        opacity: 1,
        collected: false
    }
])

// Mouse tracking
const updateMousePosition = (event) => {
    mousePosition.value = {
        x: event.clientX - 32, // Offset to center the ghost
        y: event.clientY - 32
    }
}

// Drag and drop functions
const startDrag = (item, event) => {
    if (incorrectItems.value.has(item.id)) return

    event.preventDefault()
    draggedItem.value = item

    document.addEventListener('mousemove', updateMousePosition)
    document.addEventListener('mouseup', endDrag)

    updateMousePosition(event)
}

const endDrag = () => {
    if (!draggedItem.value) return

    if (isOverDropZone.value) {
        handleDrop()
    } else {
        // Item was dropped outside the zone, animate back
        animateItemReturn(draggedItem.value)
    }

    draggedItem.value = null
    document.removeEventListener('mousemove', updateMousePosition)
    document.removeEventListener('mouseup', endDrag)
}

const handleDrop = () => {
    if (!draggedItem.value) return

    if (draggedItem.value.isCorrect) {
        // Correct answer!
        draggedItem.value.collected = true
        gameCompleted.value = true
    } else {
        // Wrong answer - add to incorrect items and animate back
        incorrectItems.value.add(draggedItem.value.id)
        animateItemReturn(draggedItem.value)

        // Make item semi-transparent
        const itemIndex = items.value.findIndex(item => item.id === draggedItem.value.id)
        if (itemIndex !== -1) {
            items.value[itemIndex].opacity = 0.4
        }
    }
}

const animateItemReturn = (item) => {
    setTimeout(() => {
        const itemElement = document.querySelector(`[data-item-id="${item.id}"]`)
        if (itemElement) {
            itemElement.style.animation = 'bounce 0.5s ease-in-out'
            setTimeout(() => {
                itemElement.style.animation = ''
            }, 500)
        }
    }, 100)
}

const onDropZoneEnter = () => {
    isOverDropZone.value = true
}

const onDropZoneLeave = () => {
    isOverDropZone.value = false
}

const continueToNext = () => {
    router.push('/')
}

onMounted(() => {
    // Prevent default drag behavior on elements
    document.addEventListener('dragstart', (e) => {
        e.preventDefault()
    })
})

onUnmounted(() => {
    document.removeEventListener('mousemove', updateMousePosition)
    document.removeEventListener('mouseup', endDrag)
})
</script>

<style scoped>
.clues-game {
    height: 100vh;
    width: 100vw;
    background: #f5f5f5;
    padding: 0;
    position: relative;
    overflow: hidden;
    box-sizing: border-box;
}

.post-it-note {
    position: absolute;
    top: 10px;
    left: 10px;
    background-image: url('/images/post-it-sand.png');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    width: 180px;
    height: 180px;
    display: flex;
    align-items: center;
    justify-content: center;
    transform: rotate(-5deg);
    z-index: 10;
}

.post-it-note h2 {
    color: #333;
    font-weight: bold;
    text-align: center;
    font-size: 20px;
    margin: 0;
}

.game-container {
    display: flex;
    height: 100vh;
    padding-top: 80px;
}

.notebook-container {
    width: 50%;
    padding: 2rem;
    height: 500px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.notebook-page {
    padding: 2rem;
    max-width: 400px;
    height: 500px;
    background-image: url('/images/notebook.png');
    line-height: 25px;
}

.notebook-page p {
    font-size: 18px;
    color: #333;
    margin: 0;
    line-height: 1.6;
}

.mugshot-container {
    width: 50%;
    padding: 2rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.mugshot-background {
    position: relative;
    width: 320px;
    height: 384px;
    border: 4px solid #000;
    background: #e5e7eb;
    margin-bottom: 2rem;
    transition: all 0.2s ease;
}

.mugshot-background.drop-zone-active {
    border-color: #2196f3;
    background-color: #e3f2fd;
}

.mugshot-image {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.pointing-hand {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 4rem;
}

.success-overlay {
    position: absolute;
    inset: 0;
    background: rgba(76, 175, 80, 0.2);
    display: flex;
    align-items: center;
    justify-content: center;
}

.success-text {
    font-size: 2rem;
    font-weight: bold;
    color: #2e7d32;
}

.size-options {
    display: flex;
    gap: 1.5rem;
}

.plastic-piece {
    cursor: grab;
    transition: transform 0.2s ease;
    z-index: 5;
    user-select: none;
}

.plastic-piece:hover:not(.collected):not(.dragging):not(.incorrect) {
    transform: scale(1.1);
}

.plastic-piece.dragging {
    opacity: 0.3;
    cursor: grabbing;
}

.plastic-piece.collected {
    opacity: 0;
    pointer-events: none;
}

.plastic-piece.incorrect {
    cursor: not-allowed;
}

.plastic-piece img {
    width: 80px;
    height: 80px;
    object-fit: contain;
    filter: drop-shadow(2px 2px 4px rgba(0, 0, 0, 0.3));
    pointer-events: none;
}

.drag-ghost {
    position: fixed;
    width: 80px;
    height: 80px;
    pointer-events: none;
    z-index: 1000;
    transform: scale(1.2);
    opacity: 0.9;
}

.drag-ghost img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    filter: drop-shadow(4px 4px 8px rgba(0, 0, 0, 0.5));
}

.bottom-elements {
    position: absolute;
    bottom: 2rem;
    left: 2rem;
    display: flex;
    gap: 2rem;
}

.green-arrow {
    width: 48px;
    height: 48px;
    background: #4caf50;
    transform: rotate(45deg);
    position: relative;
}

.green-arrow::after {
    content: '';
    position: absolute;
    inset: 8px;
    background: #66bb6a;
}



.continue-btn {
    position: absolute;
    bottom: 2rem;
    right: 2rem;
    background: white;
    border: 2px solid black;
    padding: 1rem 2rem;
    font-size: 1.25rem;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.continue-btn:hover {
    background: #f5f5f5;
}

/* Bounce animation */
@keyframes bounce {

    0%,
    20%,
    50%,
    80%,
    100% {
        transform: translateY(0);
    }

    40% {
        transform: translateY(-10px);
    }

    60% {
        transform: translateY(-5px);
    }
}

/* Responsive design */
@media (max-width: 768px) {
    .game-container {
        flex-direction: column;
        padding-top: 50px;
    }

    .notebook-container,
    .mugshot-container {
        width: 100%;
        padding: 1rem;
    }

    .mugshot-background {
        width: 280px;
        height: 336px;
    }

    .size-options {
        gap: 1rem;
    }

    .plastic-piece img {
        width: 70px;
        height: 70px;
    }
}
</style>