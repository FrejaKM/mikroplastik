<template>
    <Base postItUrl="/images/post-it-sand.png" title="Sporene"
        description="En essentiel del af efterforskningen finder sted i supermarkedet. Her gemmer nogle af de  vigtigste spor i mikroplast mysteriet sig på hylderne – uden at nogen lægger mærke til dem."
        instructions="Træk produkter, der er kendt for at indeholde mikroplast eller blive til mikroplast, når de nedbrydes, ned i kurven"
        :completed="gameCompleted" @continue="continueToNext">

    <div class="game-container">
        <!-- Shelf with products -->
        <div class="shelf-container">
            <img src="/images/final_shelves.png" alt="Shelf" class="shelf-background" />

            <!-- Products on shelf -->
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

// Products data - positioned on four shelves with realistic scaling
const products = ref([
    // TOP SHELF (25% from top) - Small items like makeup, gum, pods
    {
        id: 5,
        name: 'Makeup',
        image: '/images/products/makeup.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.5,
        position: { position: 'absolute', top: '67%', left: '50%' }
    },
    {
        id: 9,
        name: 'Gum',
        image: '/images/products/gum.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1,
        position: { position: 'absolute', top: '11%', left: '66%' }
    },
    {
        id: 4,
        name: 'Pods',
        image: '/images/products/pods.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.5,
        position: { position: 'absolute', top: '48%', left: '75%' }
    },
    {
        id: 7,
        name: 'Tea Bags',
        image: '/images/products/teabag.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1,
        position: { position: 'absolute', top: '50%', left: '35%' }
    },
    {
        id: 20,
        name: 'Candle',
        image: '/images/products/candle.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.3,
        position: { position: 'absolute', top: '48%', left: '45%' }
    },
    {
        id: 16,
        name: 'Pickles',
        image: '/images/products/pickles.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.2,
        position: { position: 'absolute', top: '29%', left: '72%' }
    },
    {
        id: 17,
        name: 'Can',
        image: '/images/products/can.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.2,
        position: { position: 'absolute', top: '31%', left: '57%' }
    },

    // SECOND SHELF (40% from top) - Medium items
    {
        id: 6,
        name: 'Toothpaste',
        image: '/images/products/toothpaste.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.3,
        position: { position: 'absolute', top: '14%', left: '55%' }
    },
    {
        id: 1,
        name: 'Sponges',
        image: '/images/products/sponges.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '27%', left: '80%' }
    },
    {
        id: 2,
        name: 'Cola',
        image: '/images/products/cola.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.7,
        position: { position: 'absolute', top: '29%', left: '42%' }
    },
    {
        id: 8,
        name: 'Chips',
        image: '/images/products/chips.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '25%', left: '15%' }
    },
    {
        id: 13,
        name: 'Milk',
        image: '/images/products/milk.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.7,
        position: { position: 'absolute', top: '26%', left: '28%' }
    },
    {
        id: 14,
        name: 'Soda Can',
        image: '/images/products/soda.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.0,
        position: { position: 'absolute', top: '30%', left: '66%' }
    },
    {
        id: 12,
        name: 'Eggs',
        image: '/images/products/eggs.png',
        containsMicroplastic: false,
        collected: false,
        scale: 2,
        position: { position: 'absolute', top: '49%', left: '60%' }
    },

    // THIRD SHELF (55% from top) - Larger items
    {
        id: 11,
        name: 'Oil',
        image: '/images/products/oil.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '25%', left: '22%' }
    },
    {
        id: 18,
        name: 'Toilet paper',
        image: '/images/products/paper.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '10%', left: '77%' }
    },
    {
        id: 19,
        name: 'Newspaper',
        image: '/images/products/newspaper.png',
        containsMicroplastic: false,
        collected: false,
        scale: 3,
        position: { position: 'absolute', top: '48%', left: '20%' }
    },
    {
        id: 15,
        name: 'Socks',
        image: '/images/products/socks.png',
        containsMicroplastic: false,
        collected: false,
        scale: 1.5,
        position: { position: 'absolute', top: '65%', left: '60%' }
    },

    // BOTTOM SHELF (70% from top) - Largest items
    {
        id: 3,
        name: 'Shoes',
        image: '/images/products/shoes.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.8,
        position: { position: 'absolute', top: '65%', left: '18%' }
    },
    {
        id: 10,
        name: 'Shorts',
        image: '/images/products/shorts.png',
        containsMicroplastic: true,
        collected: false,
        scale: 1.9,
        position: { position: 'absolute', top: '65%', left: '35%' }
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
    const maxLeft = 60 // Leave room for item width
    const minTop = 0
    const maxTop = 60 // Leave room for item height

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

const continueToNext = () => {
    router.push('/')
}

onMounted(() => {
    // Prevent default drag behavior on images
    document.addEventListener('dragstart', (e) => {
        if (e.target.tagName === 'IMG') {
            e.preventDefault()
        }
    })
})

onUnmounted(() => {
    // Clean up event listeners
    document.removeEventListener('mousemove', updateMousePosition)
    document.removeEventListener('mouseup', endDrag)
})
</script>

<style scoped>
.game-container {
    display: flex;
    justify-content: center;
    align-items: center;
    max-width: 100%;
    height: 100%;
    margin: 0;
    box-sizing: border-box;
}

.shelf-container {
    position: relative;
    width: 100%;
    max-width: 900px;
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
    transition: transform 0.2s ease;
    z-index: 5;
    user-select: none;
    width: 8%;
    /* Base size relative to shelf width */
    height: auto;
    aspect-ratio: 1;
}

.product:hover:not(.collected):not(.dragging) {
    transform: scale(1.1);
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
    filter: drop-shadow(0.2vw 0.2vw 0.4vw rgba(0, 0, 0, 0.3));
    pointer-events: none;
}

.drag-ghost {
    position: fixed;
    width: 8vw;
    height: 8vw;
    max-width: 80px;
    max-height: 80px;
    min-width: 60px;
    min-height: 60px;
    pointer-events: none;
    z-index: 1000;
    opacity: 0.9;
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
    width: 19vw;
    height: 19vw;
    max-width: 300px;
    max-height: 300px;
    min-width: 200px;
    min-height: 200px;
    right: min(450px, 30vw);
    bottom: min(150px, 10vw);
    rotate: 10deg;
    transition: transform 0.2s ease;
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
    width: 20%;
    height: 20%;
    position: absolute;
    aspect-ratio: 1;
}

.basket-item img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    filter: drop-shadow(0.1vw 0.1vw 0.2vw rgba(0, 0, 0, 0.3));
}

.score {
    position: absolute;
    bottom: -10px;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    box-sizing: border-box;
    font-size: 30px;
    color: #2c3e50;
    margin-bottom: 15px;
}

/* Bounce animation for wrong answers */
@keyframes bounce {

    0%,
    20%,
    50%,
    80%,
    100% {
        transform: translateY(0);
    }

    40% {
        transform: translateY(-20px);
    }

    60% {
        transform: translateY(-10px);
    }
}

/* Responsive design */
@media (max-width: 768px) {
    .game-container {
        flex-direction: column;
        gap: 40px;
        padding-top: 50px;
    }

    .shelf-container {
        width: 100%;
        max-width: 500px;
    }
}
</style>