<template>
    <Base 
    postItUrl="/images/post-it/post-it-pink.png"
    instructionsUrl="/images/ui/instructions_solution.png"
    notebookUrl="/images/ui/solution_notebook.png"
    title="Opklaringen"
    levelId="solution" 
    :completed="gameCompleted" 
    @continue="handleContinue" 
    @restart="resetGame">

    <div class="game-container">
        <!-- Game Items Area -->
        <div class="items-container">
            <div v-for="item in gameItems" :key="item.id" class="game-item-slot">
                <div v-if="![...goodHabitsItems.map(i => i.id), ...badHabitsItems.map(i => i.id)].includes(item.id)"
                    class="game-item" :class="{ 'shake': item.shake }" :draggable="true"
                    @dragstart="startDrag(item.id, $event)" @dragend="endDrag"
                    @touchstart="handleTouchStart(item.id, $event)" @touchmove="handleTouchMove"
                    @touchend="handleTouchEnd">
                    <img :src="item.image" :alt="item.name" />
                </div>
            </div>
        </div>

        <!-- Drop Zones -->
        <div class="drop-zones">
            <!-- Good Habits Box -->
            <div class="drop-zone good-zone" @dragover.prevent @drop="dropItem('good', $event)"
                @dragenter.prevent="handleDragEnter('good')" @dragleave="handleDragLeave('good')">
                <p>Gode vaner ✓</p>
                <div class="dropped-items">
                    <img v-for="item in goodHabitsItems" :key="`good-${item.id}`" :src="item.image" :alt="item.name"
                        :class="['dropped-item', { 'bounce': item.justAdded }]" />
                </div>
            </div>

            <!-- Bad Habits Box -->
            <div class="drop-zone bad-zone" @dragover.prevent @drop="dropItem('bad', $event)"
                @dragenter.prevent="handleDragEnter('bad')" @dragleave="handleDragLeave('bad')">
                <p>Dårlige vaner ✗</p>
                <div class="dropped-items">
                    <img v-for="item in badHabitsItems" :key="`bad-${item.id}`" :src="item.image" :alt="item.name"
                        :class="['dropped-item', { 'bounce': item.justAdded }]" />
                </div>
            </div>
        </div>
    </div>
    </Base>
</template>

<script>
import { nextTick } from 'vue';
import Base from '../components/Base.vue';

export default {
    name: 'SolutionView',
    components: {
        Base
    },
    data() {
        return {
            draggedItem: null,
            dragOverZone: null,
            touchStartPos: null,
            gameItems: [
                {
                    id: 'reusable-bottle',
                    name: 'Genbrugsflaske',
                    image: '/images/habits/bottle_metal.png',
                    category: 'good',
                    shake: false
                },
                {
                    id: 'plastic-bottle',
                    name: 'Plastikflaske',
                    image: '/images/habits/bottle_plastic.png',
                    category: 'bad',
                    shake: false
                },
                {
                    id: 'canvas-bag',
                    name: 'Stofpose',
                    image: '/images/habits/totebag.png',
                    category: 'good',
                    shake: false
                },
                {
                    id: 'plastic-bag',
                    name: 'Plastikpose',
                    image: '/images/habits/plasticbag.png',
                    category: 'bad',
                    shake: false
                },
                {
                    id: 'recycling-bin',
                    name: 'Affaldsortering',
                    image: '/images/habits/trash_empty.png',
                    category: 'good',
                    shake: false
                },
                {
                    id: 'metal-straws',
                    name: 'Metalsugerør',
                    image: '/images/habits/straws_metal.png',
                    category: 'good',
                    shake: false
                },
                {
                    id: 'plastic-straws',
                    name: 'Plastiksugerør',
                    image: '/images/habits/straws_plastic.png',
                    category: 'bad',
                    shake: false
                },
                {
                    id: 'quality-clothing',
                    name: 'Kvalitetstøj',
                    image: '/images/habits/sweater.png',
                    category: 'good',
                    shake: false
                },
                {
                    id: 'full-bin',
                    name: 'Overfyldt skraldespand',
                    image: '/images/habits/trash_full.png',
                    category: 'bad',
                    shake: false
                },
                {
                    id: 'fast-fashion',
                    name: 'Fast fashion',
                    image: '/images/habits/fleece.png',
                    category: 'bad',
                    shake: false
                }
            ],
            goodHabitsItems: [],
            badHabitsItems: []
        };
    },
    computed: {
        availableItems() {
            const placedIds = [
                ...this.goodHabitsItems.map(item => item.id),
                ...this.badHabitsItems.map(item => item.id)
            ];
            return this.gameItems.filter(item => !placedIds.includes(item.id));
        },
        correctItems() {
            return this.goodHabitsItems.length + this.badHabitsItems.length;
        },
        totalItems() {
            return this.gameItems.length;
        },
        gameCompleted() {
            return this.correctItems === this.totalItems && this.availableItems.length === 0;
        }
    },
    watch: {
        gameCompleted(newVal) {
            if (newVal) {
                this.markAsCompleted();
                this.saveProgress();
            }
        }
    },
    async mounted() {
        // Wait for DOM to be fully rendered
        await nextTick();

        // Load any saved progress
        this.loadProgress();

        // Ensure drag functionality is properly initialized
        this.$nextTick(() => {
            this.initializeDragAndDrop();
        });
    },
    methods: {
        initializeDragAndDrop() {
            // Ensure all draggable elements are properly set up
            const draggableElements = this.$el.querySelectorAll('.game-item');
            draggableElements.forEach(element => {
                element.setAttribute('draggable', 'true');
            });
        },

        startDrag(itemId, event) {
            this.draggedItem = itemId;
            if (event.dataTransfer) {
                event.dataTransfer.setData('text/plain', itemId);
                event.dataTransfer.effectAllowed = 'move';
            }
        },

        endDrag() {
            this.draggedItem = null;
            this.dragOverZone = null;
        },

        handleDragEnter(zone) {
            this.dragOverZone = zone;
        },

        handleDragLeave(zone) {
            // Small delay to prevent flickering when moving over child elements
            setTimeout(() => {
                if (this.dragOverZone === zone) {
                    this.dragOverZone = null;
                }
            }, 50);
        },

        dropItem(zone, event) {
            event.preventDefault();
            const itemId = event.dataTransfer ?
                event.dataTransfer.getData('text/plain') :
                this.draggedItem;

            const item = this.gameItems.find(i => i.id === itemId);

            if (!item) return;

            const isCorrect = (zone === 'good' && item.category === 'good') ||
                (zone === 'bad' && item.category === 'bad');

            if (isCorrect) {
                // Add bounce animation flag to the item
                const itemWithBounce = { ...item, justAdded: true };

                if (zone === 'good') {
                    this.goodHabitsItems.push(itemWithBounce);
                } else {
                    this.badHabitsItems.push(itemWithBounce);
                }

                // Remove bounce flag after animation completes
                setTimeout(() => {
                    const targetArray = zone === 'good' ? this.goodHabitsItems : this.badHabitsItems;
                    const placedItem = targetArray.find(i => i.id === item.id);
                    if (placedItem) {
                        placedItem.justAdded = false;
                    }
                }, 800); // Match animation duration

                this.saveProgress();
            } else {
                // Visual feedback for wrong placement
                this.showWrongFeedback(item);
            }

            this.draggedItem = null;
            this.dragOverZone = null;
        },

        // Touch support for mobile devices
        handleTouchStart(itemId, event) {
            this.draggedItem = itemId;
            this.touchStartPos = {
                x: event.touches[0].clientX,
                y: event.touches[0].clientY
            };
        },

        handleTouchMove(event) {
            event.preventDefault(); // Prevent scrolling
        },

        handleTouchEnd(event) {
            if (!this.draggedItem || !this.touchStartPos) return;

            const touch = event.changedTouches[0];
            const elementBelow = document.elementFromPoint(touch.clientX, touch.clientY);

            if (elementBelow) {
                const dropZone = elementBelow.closest('.drop-zone');
                if (dropZone) {
                    const zone = dropZone.classList.contains('good-zone') ? 'good' : 'bad';
                    this.dropItem(zone, { preventDefault: () => { }, dataTransfer: null });
                }
            }

            this.draggedItem = null;
            this.touchStartPos = null;
        },

        showWrongFeedback(item) {
            // Add shake animation to the original item
            const originalItem = this.gameItems.find(i => i.id === item.id);
            if (originalItem) {
                originalItem.shake = true;
                setTimeout(() => {
                    originalItem.shake = false;
                }, 600); // Match shake animation duration
            }
        },

        saveProgress() {
            const progressData = {
                goodHabitsItems: this.goodHabitsItems,
                badHabitsItems: this.badHabitsItems
            };
            localStorage.setItem('levelProgress_solution', JSON.stringify(progressData));
        },

        loadProgress() {
            const saved = localStorage.getItem('levelProgress_solution');
            if (saved) {
                const savedData = JSON.parse(saved);
                this.goodHabitsItems = savedData.goodHabitsItems || [];
                this.badHabitsItems = savedData.badHabitsItems || [];
            }
        },

        resetGame() {
            this.goodHabitsItems = [];
            this.badHabitsItems = [];
            this.draggedItem = null;
            this.dragOverZone = null;

            // Reset shake animations for all items
            this.gameItems.forEach(item => {
                item.shake = false;
            });

            // Remove progress from localStorage
            localStorage.removeItem('levelProgress_solution');

            // Remove from completed levels
            const saved = localStorage.getItem('completedLevels');
            if (saved) {
                const completedLevels = JSON.parse(saved);
                delete completedLevels['solution'];
                localStorage.setItem('completedLevels', JSON.stringify(completedLevels));
            }
        },

        markAsCompleted() {
            const saved = localStorage.getItem('completedLevels');
            const completedLevels = saved ? JSON.parse(saved) : {};
            completedLevels['solution'] = true;
            localStorage.setItem('completedLevels', JSON.stringify(completedLevels));

            // Store completion timestamp for animation detection
            localStorage.setItem(`completion_time_solution`, Date.now().toString());
        },

        handleContinue() {
            this.$router.push('/board');
        }
    }
};
</script>

<style scoped>
.game-container {
    width: 100%;
    height: 90%;
    padding: var(--padding-medium);
    padding-top: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
}

.items-container {
    padding: var(--padding-medium);
    padding-bottom: 0;
    padding-top: 3vh;
    padding-bottom: 3vh;
    width: 100%;
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
}

.game-item-slot {
    width: auto;
    height: 15vh;
    display: flex;
    align-items: center;
    justify-content: center;
}

.game-item {
    cursor: grab;
    transition: transform 0.2s ease;
    user-select: none;
}

.game-item:active {
    cursor: grabbing;
}

.game-item:hover {
    transform: scale(1.05);
}

.game-item img {
    width: auto;
    height: 15vh;
    object-fit: contain;
    pointer-events: none;
}

.drop-zones {
    display: grid;
    grid-template-columns: 1fr 1fr;
    align-items: center;
    column-gap: var(--padding-small); /* or any value you want for the space between */
}

.drop-zone.drag-over {
    transform: scale(1.02);
    filter: brightness(1.1);
}
.drop-zone {
    justify-content: center; /* Changed from center to flex-start */
    align-items: center;
    min-height: 45vh; /* Ensure it doesn't shrink */
    max-height: 45vh; /* Ensure it doesn't grow */
    min-width: 60vh;
    max-width: 6vh;
    background-color: transparent;
    padding-top: 10px;
    background-image: url('/images/ui/drop-zone.png');
    background-size: contain;
    background-repeat: no-repeat;
    transition: transform 0.2s ease, filter 0.2s ease;
    display: flex;
    column-gap: 0;
    row-gap: 0;
    flex-direction: column;
    overflow: hidden; /* Prevent content from expanding the zone */
}

.drop-zone p {
    text-align: center;
    padding: 0;
    font-size: var(--font-large);
    flex-shrink: 0; /* Prevent the title from shrinking */
}

.dropped-items {
    display: flex;
    flex-wrap: wrap;
    column-gap: 0;
    justify-content: center;
    align-items: flex-start; /* Changed to flex-start for better alignment */
    flex: 1; /* Take up remaining space */
    overflow-y: auto; /* Allow scrolling if too many items */
    overflow-x: hidden;
    padding: var(--padding-small); /* Add some padding */
    width: 100%; /* Ensure full width usage */
}
.dropped-item {
    width: auto;
    height: 15vh;;
    object-fit: contain;
    transition: transform 0.3s ease;
}

.dropped-item:hover {
    transform: scale(1.05);
}

/* Animations */
@keyframes wrongPlacement {
    0%, 100% {
        transform: translateX(0);
    }
    25% {
        transform: translateX(-5px);
    }
    75% {
        transform: translateX(5px);
    }
}

.wrong-placement {
    animation: wrongPlacement 0.3s ease-in-out;
}

/* Shake animation for wrong items (similar to CluesView) */
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

.game-item.shake {
    animation: shake 0.6s ease-in-out;
}

/* Bounce animation for correct items (similar to CluesView) */
@keyframes bounce {
    0% {
        transform: scale(0.5);
    }
    25% {
        transform: scale(1.2);
    }
    50% {
        transform: scale(0.9);
    }
    75% {
        transform: scale(1.1);
    }
    100% {
        transform: scale(1);
    }
}

.dropped-item.bounce {
    animation: bounce 0.8s ease-out;
}
</style>