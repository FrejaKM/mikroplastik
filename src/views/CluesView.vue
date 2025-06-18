<template>
    <Base 
    postItUrl="/images/post-it/post-it-sand.png" 
    instructionsUrl="/images/ui/instructions_clues.png"
    notebookUrl="/images/ui/clues_notebook.png"
    title="Sporene" 
    levelId="sporene" 
    :completed="gameCompleted" 
    @continue="continueToNext" 
    @restart="restartLevel">


    <div class="game-container">
        <div class="shelf-container">
            <img src="/images/ui/shelves.png" alt="Shelf" class="shelf-background" />

            <div v-for="product in products" :key="product.id" :class="['product', {
                'dragging': draggingProduct === product.id,
                'collected': product.collected
            }]" :style="{
                ...product.position,
                transform: `scale(${product.scale})`
            }" @mousedown="startDrag(product, $event)">
                <img :src="product.image" :alt="product.name" :class="{ 'shake': product.shake }" />
            </div>
        </div>

        <!-- Shopping basket -->
        <div class="basket-container" @mouseover="onBasketHover" @mouseleave="onBasketLeave">
            <img src="/images/ui/basket.png" alt="Shopping basket" class="basket" />
            <div class="basket-items">
                <div v-for="item in collectedItems" :key="item.id" :class="['basket-item', {
                    'bounce': item.justAdded
                }]" :style="{
                    top: '55%',
                    left: '50%',
                    transform: 'translate(-50%, -50%)'
                }">
                    <img :src="item.image" :alt="item.name" />
                </div>
            </div>
            <!-- Score and instructions -->
            <div class="game-info">
                <div class="score">{{ collectedCount }}/{{ totalCorrectItems }}</div>
            </div>
        </div>

        <!-- Dragging ghost element -->
        <div v-if="draggingProduct" class="drag-ghost" :style="{
            left: mousePosition.x + 'px',
            top: mousePosition.y + 'px'
        }">
            <img :src="getDraggingProduct()?.image" :alt="getDraggingProduct()?.name" />
        </div>
    </div>
    </Base>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import Base from '../components/Base.vue'

const router = useRouter()

// Game state
const draggingProduct = ref(null)
const collectedItems = ref([])
const mousePosition = ref({ x: 0, y: 0 })
const isOverBasket = ref(false)

const products = ref([
    // TOP SHELF 
    {
        id: 6,
        name: 'Toothpaste',
        image: '/images/products/toothpaste.png',
        containsMicroplastic: true,
        collected: false,
        shake: false,
        scale: 1.3,
        position: { position: 'absolute', top: '-1%', left: '55%' }
    },
    {
        id: 9,
        name: 'Gum',
        image: '/images/products/gum.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1,
        position: { position: 'absolute', top: '-5%', left: '65%' }
    },
    {
        id: 18,
        name: 'Toilet paper',
        image: '/images/products/paper.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.8,
        position: { position: 'absolute', top: '-6%', left: '75%' }
    },

    // SECOND SHELF 
    {
        id: 11,
        name: 'Oil',
        image: '/images/products/oil.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.8,
        position: { position: 'absolute', top: '18%', left: '22%' }
    },
    {
        id: 8,
        name: 'Chips',
        image: '/images/products/chips.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.8,
        position: { position: 'absolute', top: '18%', left: '10%' }
    },
    {
        id: 13,
        name: 'Milk',
        image: '/images/products/milk.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.7,
        position: { position: 'absolute', top: '19%', left: '28%' }
    },
    {
        id: 1,
        name: 'Sponges',
        image: '/images/products/sponges.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.8,
        position: { position: 'absolute', top: '20%', left: '85%' }
    },
    {
        id: 2,
        name: 'Cola',
        image: '/images/products/cola.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.7,
        position: { position: 'absolute', top: '21%', left: '42%' }
    },
    {
        id: 16,
        name: 'Pickles',
        image: '/images/products/pickles.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.2,
        position: { position: 'absolute', top: '22%', left: '76%' }
    },
    {
        id: 14,
        name: 'Soda Can',
        image: '/images/products/soda.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.0,
        position: { position: 'absolute', top: '22%', left: '66%' }
    },
    {
        id: 17,
        name: 'Can',
        image: '/images/products/can.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.2,
        position: { position: 'absolute', top: '24%', left: '57%' }
    },

    // THIRD SHELF 
    {
        id: 19,
        name: 'Newspaper',
        image: '/images/products/newspaper.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 3,
        position: { position: 'absolute', top: '48%', left: '15%' }
    },
    {
        id: 7,
        name: 'Tea Bags',
        image: '/images/products/teabag.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1,
        position: { position: 'absolute', top: '49%', left: '30%' }
    },
    {
        id: 20,
        name: 'Candle',
        image: '/images/products/candle.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.3,
        position: { position: 'absolute', top: '47%', left: '42%' }
    },
    {
        id: 12,
        name: 'Eggs',
        image: '/images/products/eggs.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 2,
        position: { position: 'absolute', top: '48%', left: '60%' }
    },
    {
        id: 4,
        name: 'Pods',
        image: '/images/products/pods.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.5,
        position: { position: 'absolute', top: '47%', left: '80%' }
    },

    // BOTTOM SHELF 
    {
        id: 15,
        name: 'Socks',
        image: '/images/products/socks.png',
        containsMicroplastic: false,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.5,
        position: { position: 'absolute', top: '70%', left: '64%' }
    },
    {
        id: 10,
        name: 'Shorts',
        image: '/images/products/shorts.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.9,
        position: { position: 'absolute', top: '70%', left: '32%' }
    },
    {
        id: 3,
        name: 'Shoes',
        image: '/images/products/shoes.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.8,
        position: { position: 'absolute', top: '72%', left: '12%' }
    },
    {
        id: 5,
        name: 'Makeup',
        image: '/images/products/makeup.png',
        containsMicroplastic: true,
        collected: false,
        wrongFlash: false,
        shake: false,
        showWrongIcon: false,
        scale: 1.5,
        position: { position: 'absolute', top: '74%', left: '50%' }
    }
])

// Computed properties
const totalCorrectItems = computed(() =>
    products.value.filter(p => p.containsMicroplastic).length
)

const collectedCount = computed(() => collectedItems.value.length)

const gameCompleted = computed(() =>
    collectedCount.value === totalCorrectItems.value
)

// Helper function to get currently dragging product
const getDraggingProduct = () => {
    return products.value.find(p => p.id === draggingProduct.value)
}

// Mouse tracking
const updateMousePosition = (event) => {
    mousePosition.value = {
        x: event.clientX - 40, // Offset to center the ghost image
        y: event.clientY - 40
    }
}

// Drag and drop functions
const startDrag = (product, event) => {
    if (product.collected) return

    event.preventDefault()
    draggingProduct.value = product.id

    // Add mouse move listener
    document.addEventListener('mousemove', updateMousePosition)
    document.addEventListener('mouseup', endDrag)

    // Set initial mouse position
    updateMousePosition(event)
}

const endDrag = (event) => {
    if (!draggingProduct.value) return

    // Check if dropped on basket
    if (isOverBasket.value) {
        handleDrop()
    }

    // Clean up
    draggingProduct.value = null
    document.removeEventListener('mousemove', updateMousePosition)
    document.removeEventListener('mouseup', endDrag)
}

const onBasketHover = () => {
    isOverBasket.value = true
}

const onBasketLeave = () => {
    isOverBasket.value = false
}

const handleDrop = () => {
    const product = getDraggingProduct()

    if (!product || product.collected) return

    if (product.containsMicroplastic) {
        product.collected = true

        // Add bounce animation flag
        const itemWithBounce = { ...product, justAdded: true }
        collectedItems.value.push(itemWithBounce)

        // Remove bounce flag after animation completes
        setTimeout(() => {
            const item = collectedItems.value.find(item => item.id === product.id)
            if (item) {
                item.justAdded = false
            }
        }, 800) // Match animation duration

        saveProgress()
        console.log('Correct!', product.name)
    } else {
        // Wrong! Show multiple feedback types
        showWrongFeedback(product)
        console.log('Wrong!', product.name)
    }
}

const showWrongFeedback = (product) => {
    // Shake the product
    product.shake = true
    setTimeout(() => {
        product.shake = false
    }, 600)
}

// Progress saving and loading
const saveProgress = () => {
    const progressData = {
        collectedItems: collectedItems.value,
        collectedProducts: products.value
            .filter(p => p.collected)
            .map(p => p.id)
    }
    localStorage.setItem('levelProgress_sporene', JSON.stringify(progressData))
}

const loadProgress = () => {
    const saved = localStorage.getItem('levelProgress_sporene')
    if (saved) {
        const savedData = JSON.parse(saved)
        collectedItems.value = savedData.collectedItems || []

        // Mark collected products
        savedData.collectedProducts?.forEach(productId => {
            const product = products.value.find(p => p.id === productId)
            if (product) {
                product.collected = true
            }
        })
    }
}

// Level management functions
const restartLevel = () => {
    // Reset game state
    collectedItems.value = []
    products.value.forEach(product => {
        product.collected = false
        product.shake = false
    })

    // Remove progress from localStorage
    localStorage.removeItem('levelProgress_sporene')

    // Remove from completed levels
    const saved = localStorage.getItem('completedLevels')
    if (saved) {
        const completedLevels = JSON.parse(saved)
        delete completedLevels['sporene']
        localStorage.setItem('completedLevels', JSON.stringify(completedLevels))
    }
}

const continueToNext = () => {
    router.push('/board')
}

onMounted(() => {
    // Load saved progress when component mounts
    loadProgress()

    // Prevent default drag behavior on images
    document.addEventListener('dragstart', (e) => {
        if (e.target.tagName === 'IMG') {
            e.preventDefault()
        }
    })
})

onUnmounted(() => {
    document.removeEventListener('mousemove', updateMousePosition)
    document.removeEventListener('mouseup', endDrag)
})
</script>

<style scoped>
.shelf-container {
    width: 95vh;
    position: relative;
    height: auto;
}

.shelf-background {
    width: 100%;
    height: 80%;
    object-fit: contain;
}

.product {
    cursor: grab;
    z-index: 5;
    user-select: none;
    width: clamp(40px, 6vw, 80px);
    height: auto;
    aspect-ratio: 1;
    position: relative;
    transition: all 0.2s ease;
}

.product.dragging {
    opacity: 0;
    cursor: grabbing;
}

.product.collected {
    opacity: 0;
    pointer-events: none;
}

/* Wrong feedback animations */
.product img.shake {
    animation: shake 0.6s ease-in-out;
}

@keyframes shake {
    0%, 100% {
        transform: translateX(0);
    }
    10%, 30%, 50%, 70%, 90% {
        transform: translateX(-4px) rotate(-1deg);
    }
    20%, 40%, 60%, 80% {
        transform: translateX(4px) rotate(1deg);
    }
}

.product img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    pointer-events: none;
}

/* Wrong icon overlay */
.wrong-icon {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 40%;
    height: 40%;
    background: rgba(255, 0, 0, 0.9);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    animation: wrongIconPop 0.8s ease-out forwards;
    z-index: 10;
}

@keyframes wrongIconPop {
    0% {
        transform: translate(-50%, -50%) scale(0);
        opacity: 0;
    }
    30% {
        transform: translate(-50%, -50%) scale(1.2);
        opacity: 1;
    }
    70% {
        transform: translate(-50%, -50%) scale(1);
        opacity: 1;
    }
    100% {
        transform: translate(-50%, -50%) scale(0);
        opacity: 0;
    }
}

.drag-ghost {
    position: fixed;
    width: clamp(40px, 6vw, 80px);
    height: clamp(40px, 6vw, 80px);
    pointer-events: none;
    z-index: 1000;
    opacity: 1;
    aspect-ratio: 1;
}

.drag-ghost img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    filter: drop-shadow(0.4vw 0.4vw 0.8vw rgba(0, 0, 0, 0.5));
}

.basket-container {
    position: absolute;
    height: 20vh;
    width: 16vw;
    right: 7vw;
    bottom: 20vh;
    rotate: 10deg;
}

.basket-container:hover {
    transform: scale(1.02);
}

.basket {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.basket-items {
    position: absolute;
    top: 5%;
    left: 5%;
    width: 90%;
    height: 90%;
    overflow: hidden;
}

.basket-item {
    position: absolute;
    width: 60%;
    height: auto;
    aspect-ratio: 1;
}

.basket-item img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    background-color: transparent;
}

.basket-item.bounce {
    animation: bounce 0.8s ease-out;
}

@keyframes bounce {
    0% {
        transform: translate(-50%, -50%) scale(0.5);
    }
    25% {
        transform: translate(-50%, -50%) scale(1.2);
    }
    50% {
        transform: translate(-50%, -50%) scale(0.9);
    }
    75% {
        transform: translate(-50%, -50%) scale(1.1);
    }
    100% {
        transform: translate(-50%, -50%) scale(1);
    }
}

.score {
    font-family: 'Coming Soon', cursive;
    position: absolute;
    bottom: -4vh;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    font-size: var(--font-large);
    color: #000000;
}
</style>