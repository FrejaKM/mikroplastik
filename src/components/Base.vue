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
        },
        goToBoard() {
            this.$router.push('/board')
        }
    }
};
</script>

<template>
    <div class="level-container">
        <!-- Post-it note -->
        <div class="post-it" :style="{ backgroundImage: `url(${postItUrl})` }">
            <div class="headline">{{ title }}</div>
        </div>

        <!-- Notesbog -->
        <div class="notebook" style="background-image: url('/images/bright_notebook.png')">
            <div class="description" v-html="description"></div>
        </div>

        <!-- Spillets indhold -->
        <div class="game-content">
            <!-- Navigation buttons inside game content -->
            <div class="game-buttons">
                <button class="back-to-board-button" @click="goToBoard">
                    Tilbage til opslagstavlen
                </button>
                <button class="level-restart-button" @click="$emit('restart')">
                    Start forfra
                </button>
            </div>

            <div class="game-main">
                <slot></slot>
            </div>

            <!-- Instruktioner -->
            <div class="instructions">
                {{ instructions }}
            </div>
        </div>

        <!-- Fortsæt-knap -->
        <button v-if="completed" class="continue-button" @click="handleContinue"></button>
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
    color: #000000;
    font-size: 2.2vw;
    font-family: 'Courier Prime', monospace;
    font-weight: 400;
    text-align: center;
    width: 65%;
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

.description {
    width: 80%;
    height: 90%;
    padding: 15px;
    padding-top: 30px;
    font-size: 1.1vw;
    text-align: left;
    font-family: 'Courier Prime', monospace;
    font-weight: 400;
    color: black;
    box-sizing: border-box;
    line-height: 1.3;
}

/* Spilindhold */
.game-content {
    padding: 20px;
    grid-column: 1 / 4;
    grid-row: 1 / 7;
    background-color: #eef5ef;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    overflow: hidden;
    position: relative;
}

/* Game buttons container */
.game-buttons {
    position: absolute;
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-between;
    padding: 0 20px;
    z-index: 100;
}

/* Back to board button (top left) */
.back-to-board-button {
    background: #ffffff;
    color: rgb(0, 0, 0);
    border: 2px solid #5b5b5b;
    box-shadow: 3px 6px 5px rgba(0, 0, 0, 0.3);
    padding: 10px 15px;
    font-size: 20px;
    cursor: pointer;
    transition: all 0.2s ease;
}

.back-to-board-button:hover {
    transform: scale(1.05);
}

/* Level restart button (top right) */
.level-restart-button {
    background: #ffffff;
    color: rgb(0, 0, 0);
    border: 2px solid #5b5b5b;
    box-shadow: 3px 6px 5px rgba(0, 0, 0, 0.3);
    padding: 10px 15px;
    font-size: 20px;
    cursor: pointer;
    transition: all 0.2s ease;
}

.level-restart-button:hover {
    transform: scale(1.05);
}

.game-main {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    padding-top: 60px;
    /* Add space for the buttons */
}

/* Instruktionstekst */
.instructions {
    background: #ffffff;
    color: rgb(0, 0, 0);
    border: 2px solid #000;
    padding: 20px 20px;
    font-size: 25px;
    text-align: center;
    width: 100%;
    box-sizing: border-box;
}

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

.continue-button {
    grid-column: 4 / 5;
    grid-row: 6 / 7;
    background-image: url('/images/arrow.png');
    background-size: 60%;
    background-color: transparent;
    border: none;
    background-repeat: no-repeat;
    background-position: center;
    cursor: pointer;
    animation: pulse 2s ease-in-out infinite;
    opacity: 0.8;
}

.continue-button:hover {
    opacity: 1;
}
</style>