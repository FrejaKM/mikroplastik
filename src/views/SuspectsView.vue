<template>
  <Base postItUrl="/images/post-it-green.png" title="Ofrene"
    description="Efterforskningen starter i supermarkedet. Her gemmer nogle af de værste mikroplast-syndere sig på hylderne – uden at nogen lægger mærke til dem."
    instructions="Træk produkter, der er kendt for at indeholde mikroplast eller blive til mikroplast, når de nedbrydes, ned i kurven"
    :completed="gameCompleted" @continue="continueToNext">

  <div class="game-container">
    <!-- Shelf with products -->
    <div class="shelf-container">
      <img src="/images/horizontal_shelf.png" alt="Shelf" class="shelf-background" />

      <!-- Products on shelf -->
      <div v-for="product in products" :key="product.id" :class="['product', {
        'dragging': draggingProduct === product.id,
        'collected': product.collected
      }]" :style="product.position" @mousedown="startDrag(product, $event)">
        <img :src="product.image" :alt="product.name" />
      </div>
    </div>

    <!-- Shopping basket -->
    <div class="basket-container" @mouseup="dropProduct($event)" @mouseover="onBasketHover" @mouseleave="onBasketLeave">
      <img src="/images/basket.png" alt="Shopping basket" class="basket" />
      <div class="basket-items">
        <div v-for="item in collectedItems" :key="item.id" class="basket-item">
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

// Products data - define which products contain microplastics
const products = ref([
  {
    id: 1,
    name: 'Bananas',
    image: '/images/products/bananas.png',
    containsMicroplastic: false,
    collected: false,
    position: { position: 'absolute', top: '20%', left: '15%' }
  },
  {
    id: 2,
    name: 'Cola',
    image: '/images/products/cola.png',
    containsMicroplastic: true,
    collected: false,
    position: { position: 'absolute', top: '55%', right: '35%' }
  },
  {
    id: 3,
    name: 'Shoes',
    image: '/images/products/shoes.png',
    containsMicroplastic: true,
    collected: false,
    position: { position: 'absolute', top: '35%', left: '10%' }
  },
  {
    id: 4,
    name: 'Olive Oil',
    image: '/images/products/olive-oil.png',
    containsMicroplastic: false,
    collected: false,
    position: { position: 'absolute', top: '35%', left: '35%' }
  },
  {
    id: 5,
    name: 'Cleaning Product',
    image: '/images/products/cleaning.png',
    containsMicroplastic: true,
    collected: false,
    position: { position: 'absolute', top: '35%', right: '15%' }
  },
  {
    id: 6,
    name: 'Plastic Containers',
    image: '/images/products/containers.png',
    containsMicroplastic: true,
    collected: false,
    position: { position: 'absolute', top: '50%', left: '20%' }
  },
  {
    id: 7,
    name: 'Tea Bags',
    image: '/images/products/tea.png',
    containsMicroplastic: true,
    collected: false,
    position: { position: 'absolute', top: '50%', right: '30%' }
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
    // Correct! Add to basket
    product.collected = true
    collectedItems.value.push(product)
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
  height: 200%;
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
}

.product:hover:not(.collected):not(.dragging) {
  transform: scale(1.1);
}

.product.dragging {
  opacity: 0.3;
  cursor: grabbing;
}

.product.collected {
  opacity: 0;
  pointer-events: none;
}

.product img {
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

.basket-container {
  position: absolute;
  width: 300px;
  height: 300px;
  right: 450px;
  bottom: 150px;
  rotate: 10deg;
  transition: transform 0.2s ease;
}

.basket-container:hover {
  transform: scale(1.02);
  rotate: 8deg;
}

.basket {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.basket-items {
  position: absolute;
  top: 30%;
  left: 30%;
  width: 40%;
  height: 40%;
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  justify-content: center;
  align-items: center;
}

.basket-item {
  width: 40px;
  height: 40px;
}

.basket-item img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.score {
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  box-sizing: border-box;
  font-size: 36px;
  font-weight: bold;
  color: #2c3e50;
  margin-bottom: 15px;
}

.detective-note {
  background: rgba(255, 255, 255, 0.9);
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  max-width: 100%;
}

.detective-note p {
  margin: 0 0 10px 0;
  font-size: 14px;
  line-height: 1.4;
  color: #333;
}

.detective-note p:last-child {
  margin-bottom: 0;
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