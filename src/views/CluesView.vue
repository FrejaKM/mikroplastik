<template>
    <Base postItUrl="/images/post-it-sand.png" title="Sporene"
        description="En essentiel del af efterforskningen finder sted i supermarkedet. Her gemmer nogle af de vigtigste spor i mikroplast mysteriet sig på hylderne – uden at nogen lægger mærke til dem."
        instructions="Træk produkter, der er kendt for at indeholde mikroplast eller blive til mikroplast, når de nedbrydes, ned i kurven"
        levelId="sporene" :completed="gameCompleted" @continue="continueToNext" @restart="restartLevel">

    <div class="game-container">
        <div class="shelf-container">
            <img src="/images/hylde.png" alt="Shelf" class="shelf-background" />

            <div v-for="product in products" :key="product.id" :class="['product', {
                'dragging': draggingProduct === product.id,
                'collected': product.collected
            }]" :style="{
                ...product.position,
                transform: `scale(${product.scale})`
            }" @mousedown="startDrag(product, $event)">
                <img :src="product.image" :alt="product.name" />
            </div>
        </div>

        <!-- Shopping basket -->
        <div class="basket-container" @mouseup="dropProduct($event)" @mouseover="onBasketHover"
            @mouseleave="onBasketLeave">
            <img src="/images/basket.png" alt="Shopping basket" class="basket" />
            <div class="basket-items">
                <div v-for="item in collectedItems" :key="item.id" class="basket-item" :style="{
                    ...item.basketPosition,
                    transform: `${item.basketPosition.transform} scale(${item.scale})`
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
    // TOP SHELF (10-14% from top)
    {
        id: 6,
        name: 'Toothpaste',
        image: '/images/products/toothpaste.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.3,
        position: { position: 'absolute', top: '8%', left: '60%' }
    },
    {
        id: 9,
        name: 'Gum',
        image: '/images/products/gum.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1,
        position: { position: 'absolute', top: '4%', left: '70%' }
    },
    {
        id: 18,
        name: 'Toilet paper',
        image: '/images/products/paper.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '3%', left: '82%' }
    },

    // SECOND SHELF (25-31% from top)
    {
        id: 11,
        name: 'Oil',
        image: '/images/products/oil.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '22%', left: '22%' }
    },
    {
        id: 8,
        name: 'Chips',
        image: '/images/products/chips.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '22%', left: '10%' }
    },
    {
        id: 13,
        name: 'Milk',
        image: '/images/products/milk.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.7,
        position: { position: 'absolute', top: '23%', left: '28%' }
    },
    {
        id: 1,
        name: 'Sponges',
        image: '/images/products/sponges.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '23%', left: '85%' }
    },
    {
        id: 2,
        name: 'Cola',
        image: '/images/products/cola.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.7,
        position: { position: 'absolute', top: '25%', left: '42%' }
    },
    {
        id: 16,
        name: 'Pickles',
        image: '/images/products/pickles.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.2,
        position: { position: 'absolute', top: '25%', left: '76%' }
    },
    {
        id: 14,
        name: 'Soda Can',
        image: '/images/products/soda.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.0,
        position: { position: 'absolute', top: '26%', left: '66%' }
    },
    {
        id: 17,
        name: 'Can',
        image: '/images/products/can.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.2,
        position: { position: 'absolute', top: '28%', left: '57%' }
    },

    // THIRD SHELF (48-50% from top)
    {
        id: 19,
        name: 'Newspaper',
        image: '/images/products/newspaper.png',
        containsMicroplastic: false,
        collected: false,
        scale: 3,
        position: { position: 'absolute', top: '48%', left: '15%' }
    },
    {
        id: 7,
        name: 'Tea Bags',
        image: '/images/products/teabag.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1,
        position: { position: 'absolute', top: '49%', left: '30%' }
    },
    {
        id: 20,
        name: 'Candle',
        image: '/images/products/candle.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.3,
        position: { position: 'absolute', top: '47%', left: '42%' }
    },
    {
        id: 12,
        name: 'Eggs',
        image: '/images/products/eggs.png',
        containsMicroplastic: false,
        collected: false,
        scale: 2,
        position: { position: 'absolute', top: '48%', left: '60%' }
    },
    {
        id: 4,
        name: 'Pods',
        image: '/images/products/pods.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.5,
        position: { position: 'absolute', top: '47%', left: '80%' }
    },


    // BOTTOM SHELF (66-69% from top)
    {
        id: 15,
        name: 'Socks',
        image: '/images/products/socks.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.5,
        position: { position: 'absolute', top: '66%', left: '64%' }
    },
    {
        id: 10,
        name: 'Shorts',
        image: '/images/products/shorts.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.9,
        position: { position: 'absolute', top: '67%', left: '32%' }
    },
    {
        id: 3,
        name: 'Shoes',
        image: '/images/products/shoes.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '68%', left: '12%' }
    },
    {
        id: 5,
        name: 'Makeup',
        image: '/images/products/makeup.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.5,
        position: { position: 'absolute', top: '69%', left: '50%' }
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

// Function to generate random position in basket
const generateRandomBasketPosition = (itemIndex) => {
    // Generate random position within the basket area
    // Use percentage-based positioning for better distribution
    const minLeft = 0
    const maxLeft = 100 // Leave room for item width
    const minTop = 0
    const maxTop = 100 // Leave room for item height

    const randomLeft = minLeft + Math.random() * (maxLeft - minLeft)
    const randomTop = minTop + Math.random() * (maxTop - minTop)

    // Add slight rotation for more natural look
    const randomRotation = (Math.random() - 0.5) * 30 // -15 to +15 degrees

    return {
        position: 'absolute',
        left: `${randomLeft}%`,
        top: `${randomTop}%`,
        transform: `rotate(${randomRotation}deg)`,
        zIndex: itemIndex + 10 // Latest items have higher z-index
    }
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
    } else {
        // Not dropped on basket, animate return
        const product = getDraggingProduct()
        if (product && !product.containsMicroplastic) {
            animateProductReturn(product)
        }
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

const dropProduct = (event) => {
    // This is now handled by endDrag when mouse is over basket
}

const handleDrop = () => {
    const product = getDraggingProduct()

    if (!product || product.collected) return

    if (product.containsMicroplastic) {
        // Correct! Add to basket with random position
        product.collected = true
        const productWithBasketPosition = {
            ...product,
            basketPosition: generateRandomBasketPosition(collectedItems.value.length)
        }
        collectedItems.value.push(productWithBasketPosition)
        saveProgress() // Save progress after successful collection
        console.log('Correct!', product.name)
    } else {
        // Wrong! Product should bounce back
        animateProductReturn(product)
        console.log('Wrong!', product.name)
    }
}

const animateProductReturn = (product) => {
    // Find the product element and animate it
    setTimeout(() => {
        const productElements = document.querySelectorAll('.product')
        const productElement = productElements[product.id - 1]
        if (productElement) {
            productElement.style.animation = 'bounce 0.5s ease-in-out'
            setTimeout(() => {
                productElement.style.animation = ''
            }, 500)
        }
    }, 100)
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
    position: relative;
    width: 100%;
    max-width: 850px;
    height: auto;
    aspect-ratio: 4 / 3;
}

.shelf-background {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.product {
    cursor: grab;
    z-index: 5;
    user-select: none;
    width: 8%;
    height: auto;
    aspect-ratio: 1;
}

.product.dragging {
    opacity: 0;
    cursor: grabbing;
}

.product.collected {
    opacity: 0;
    pointer-events: none;
}

.product img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    pointer-events: none;
}

.drag-ghost {
    position: fixed;
    width: 8vw;
    height: 8vw;
    max-width: 200px;
    max-height: 200px;
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
    width: 16vw;
    height: 16vw;
    max-width: 300px;
    max-height: 300px;
    min-width: 200px;
    min-height: 200px;
    right: 32vw;
    bottom: 19vh;
    rotate: 10deg;
    aspect-ratio: 1;
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
    top: 20%;
    left: 20%;
    width: 60%;
    height: 60%;
    overflow: hidden;
}

.basket-item {
    width: 45%;
    height: 45%;
    position: absolute;
    aspect-ratio: 1;
}

.basket-item img {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.score {
    position: absolute;
    bottom: -5px;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    font-size: 30px;
    color: #000000;
}
</style>