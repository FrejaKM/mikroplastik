<script>
export default {
    name: "Base",
    props: {
        postItUrl: { type: String, required: true },
        title: { type: String, required: true },
        description: { type: String, required: true },
        instructions: { type: String, required: true },
        completed: { type: Boolean, default: false },
        levelId: { type: String, required: true } // Add levelId prop
    },
    mounted() {
        // Check if level was already completed
        this.checkCompletionStatus()
    },
    methods: {
        checkCompletionStatus() {
            const saved = localStorage.getItem('completedLevels')
            if (saved) {
                const completedLevels = JSON.parse(saved)
                if (completedLevels[this.levelId]) {
                    this.$emit('update:completed', true)
                }
            }
        },
        handleContinue() {
            // Mark level as completed
            this.markLevelCompleted()

            // Emit completion event for the board to listen to
            window.dispatchEvent(new CustomEvent('levelCompleted', {
                detail: { levelId: this.levelId }
            }))

            // Navigate back to board
            this.$router.push('/board')
        },
        markLevelCompleted() {
            const saved = localStorage.getItem('completedLevels')
            const completedLevels = saved ? JSON.parse(saved) : {}
            completedLevels[this.levelId] = true
            localStorage.setItem('completedLevels', JSON.stringify(completedLevels))
        }
    }
};
</script>

<template>
    <div class="level-container">
        <!-- Start Over Button for individual level -->
        <button v-if="completed" class="level-restart-button" @click="$emit('restart')">
            Start forfra
        </button>

        <!-- Post-it note -->
        <div class="post-it" :style="{ backgroundImage: `url(${postItUrl})` }">
            <div class="headline">{{ title }}</div>
        </div>

        <!-- Notesbog -->
        <div class="notebook" style="background-image: url('/images/bright_notebook.png')">
            <div class="info-text">{{ description }}</div>
        </div>

        <!-- Spillets indhold -->
        <div class="game-content">
            <slot></slot>

            <!-- Instruktioner -->
            <div class="instructions">
                {{ instructions }}
            </div>
        </div>

        <!-- Fortsæt-knap -->
        <button v-if="completed" class="continue-button" @click="handleContinue">
            Fortsæt →
        </button>
    </div>
</template>

<style scoped>
.level-container {
    display: grid;
    width: 100vw;
    height: 100vh;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(6, 1fr);
    gap: 20px;
    background-image: url('/images/cork-board.png');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    padding: 20px;
    box-sizing: border-box;
    overflow: hidden;
}

/* Level restart button */
.level-restart-button {
    position: absolute;
    top: 20px;
    left: 20px;
    background: #ff8c00;
    color: white;
    border: 2px solid #000;
    border-radius: 8px;
    padding: 8px 16px;
    font-size: 12px;
    font-weight: bold;
    cursor: pointer;
    z-index: 1000;
    transition: background-color 0.2s ease;
}

.level-restart-button:hover {
    background: #ffa500;
}

/* Post-it */
.post-it {
    grid-column: 4 / 5;
    grid-row: 1 / 3;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    display: flex;
    align-items: flex-start;
    justify-content: center;
    padding-top: 30%;
    overflow: hidden;
    min-height: 0;
    min-width: 0;
    rotate: 10deg;
}

.headline {
    width: 70%;
    font-size: 32px;
    text-align: center;
    color: black;
    font-weight: regular;
    line-height: 1.2;
}

/* Notesbog */
.notebook {
    grid-column: 4 / 5;
    grid-row: 3 / 6;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    display: flex;
    align-items: flex-start;
    justify-content: center;
    padding-top: 20%;
    overflow: hidden;
}

.info-text {
    width: 80%;
    height: 90%;
    padding: 3vh;
    padding-top: 1vh;
    font-size: 1.1vw;
    text-align: left;
    color: black;
    overflow-y: auto;
    overflow-x: hidden;
    box-sizing: border-box;
    line-height: 1.3;
}

/* Spilindhold */
.game-content {
    grid-column: 1 / 4;
    grid-row: 1 / 7;
    border: 2px solid #000000;
    background-color: #eef5ef;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    overflow: hidden;
}

/* Instruktionstekst */
.instructions {
    background-color: #eef5ef;
    font-size: 30px;
    margin: 30px 30px;
    margin-top: 0px;
    display: flex;
    text-align: center;
    border: #000000 2px solid;
    align-items: center;
    overflow: hidden;
    box-sizing: border-box;
}

/* Fortsæt-knap */
.continue-button {
    grid-column: 4 / 5;
    grid-row: 6 / 7;
    border: 2px solid black;
    border-radius: 10px;
    background-color: #4CAF50;
    color: white;
    cursor: pointer;
    font-size: 32px;
    padding: 8px 18px;
    font-weight: bold;
}

.continue-button:hover {
    scale: 1.02;
    background-color: #45a049;
}
</style>