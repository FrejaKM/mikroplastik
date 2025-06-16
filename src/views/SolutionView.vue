<template>
    <Base postItUrl="/images/post-it-pink.png" title="Opklaringen"
        description="Sagen er næsten opklaret. Du har identificeret og fundet mikroplasten, fulgt dens vej ud mod havet og set, hvem det går ud over. <br> <br> Men én vigtig brik mangler: Hvordan forhindrer vi, at sagen gentager sig?"
        instructions="Ved hjælp af gode vaner kan vi alle være med til at stoppe mikroplast fra at ende i havet. Sorter tingene i gode og dårlige vaner."
        levelId="solution" :completed="gameCompleted" @continue="handleContinue">

    <div class="game-container">
        <!-- Game Items Area -->
        <div class="items-container">
            <div v-for="item in gameItems" :key="item.id" class="game-item-slot">
                <div v-if="![...goodHabitsItems.map(i => i.id), ...badHabitsItems.map(i => i.id)].includes(item.id)"
                    class="game-item" draggable="true" @dragstart="startDrag(item.id, $event)" @dragend="endDrag">
                    <img :src="item.image" :alt="item.name" />
                </div>

            </div>
        </div>

        <!-- Drop Zones -->
        <div class="drop-zones">
            <!-- Good Habits Box -->
            <div class="drop-zone" @dragover.prevent @drop="dropItem('good', $event)">
                <p>Gode vaner ✓</p>
                <div class="dropped-items">
                    <img v-for="item in goodHabitsItems" :key="`good-${item.id}`" :src="item.image" :alt="item.name"
                        class="dropped-item" />
                </div>
            </div>

            <!-- Bad Habits Box -->
            <div class="drop-zone" @dragover.prevent @drop="dropItem('bad', $event)">
                <p>Dårlige vaner ✗</p>
                <div class="dropped-items">
                    <img v-for="item in badHabitsItems" :key="`bad-${item.id}`" :src="item.image" :alt="item.name"
                        class="dropped-item" />
                </div>
            </div>
        </div>
    </div>
    </Base>
</template>

<script>
import Base from '../components/Base.vue';

export default {
    name: 'SolutionView',
    components: {
        Base
    },
    data() {
        return {
            draggedItem: null,
            gameItems: [
                {
                    id: 'reusable-bottle',
                    name: 'Genbrugsflaske',
                    image: '/images/habits/bottle_metal.png',
                    category: 'good'
                },
                {
                    id: 'plastic-bottle',
                    name: 'Plastikflaske',
                    image: '/images/habits/bottle_plastic.png',
                    category: 'bad'
                },
                {
                    id: 'canvas-bag',
                    name: 'Stofpose',
                    image: '/images/habits/totebag.png',
                    category: 'good'
                },
                {
                    id: 'plastic-bag',
                    name: 'Plastikpose',
                    image: '/images/habits/plasticbag.png',
                    category: 'bad'
                },
                {
                    id: 'recycling-bin',
                    name: 'Affaldsortering',
                    image: '/images/habits/trash_empty.png',
                    category: 'good'
                },
                {
                    id: 'metal-straws',
                    name: 'Metalsugerør',
                    image: '/images/habits/straws_metal.png',
                    category: 'good'
                },
                {
                    id: 'plastic-straws',
                    name: 'Plastiksugerør',
                    image: '/images/habits/straws_plastic.png',
                    category: 'bad'
                },
                {
                    id: 'quality-clothing',
                    name: 'Kvalitetstøj',
                    image: '/images/habits/sweater.png',
                    category: 'good'
                },
                {
                    id: 'full-bin',
                    name: 'Overfyldt skraldespand',
                    image: '/images/habits/trash_full.png',
                    category: 'bad'
                },
                {
                    id: 'fast-fashion',
                    name: 'Fast fashion',
                    image: '/images/habits/fleece.png',
                    category: 'bad'
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
            }
        }
    },
    mounted() {
    },
    methods: {
        startDrag(itemId, event) {
            this.draggedItem = itemId;
            event.dataTransfer.setData('text/plain', itemId);
        },
        endDrag() {
            this.draggedItem = null;
        },
        dropItem(zone, event) {
            event.preventDefault();
            const itemId = event.dataTransfer.getData('text/plain');
            const item = this.gameItems.find(i => i.id === itemId);

            if (!item) return;

            const isCorrect = (zone === 'good' && item.category === 'good') ||
                (zone === 'bad' && item.category === 'bad');

            if (isCorrect) {
                if (zone === 'good') {
                    this.goodHabitsItems.push(item);
                } else {
                    this.badHabitsItems.push(item);
                }
            }

            this.draggedItem = null;
        },

        resetGame() {
            this.goodHabitsItems = [];
            this.badHabitsItems = [];
            this.draggedItem = null;
        },
        markAsCompleted() {
            const saved = localStorage.getItem('completedLevels');
            const completedLevels = saved ? JSON.parse(saved) : {};
            completedLevels['solution'] = true;
            localStorage.setItem('completedLevels', JSON.stringify(completedLevels));
        },
        handleContinue() {
            console.log('Fortsæt!');
        }
    }
};
</script>

<style scoped>
.game-container {
    width: 1000px;
    height: 90%;
    padding-top: 40px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.items-container {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    margin-top: 10px;
}

.game-item-slot {
    width: auto;
    height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.game-item {
    cursor: grab;
}

.game-item img {
    width: auto;
    height: 120px;
    object-fit: contain;
}

.drop-zones {
    display: grid;
    grid-template-columns: 1fr 1fr;
    justify-content: center;
    align-items: center;
    gap: 20px;
    margin-bottom: 20px;
}

.drop-zone {
    justify-content: center;
    align-items: center;
    height: 280px;
    padding: 40px;
    background-color: transparent;
    background-image: url('/images/drop-zone.png');
    background-size: contain;
    background-repeat: no-repeat;
}

.drop-zone p {
    text-align: center;
    margin: 0 0 10px 0;
    font-size: 25px;
}

.dropped-items {
    display: flex;
    flex-wrap: wrap;
    column-gap: 40px;
    justify-content: center;
    align-items: center;
}

.dropped-item {
    width: auto;
    height: 120px;
    object-fit: contain;
}
</style>