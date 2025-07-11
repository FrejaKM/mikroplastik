<script>
export default {
    name: "Base",
    props: {
        postItUrl: { type: String, required: true },
        notebookUrl: { type: String, required: true },
        instructionsUrl: { type: String, required: true },
        title: { type: String, required: true },
        completed: { type: Boolean, default: false },
        levelId: { type: String, required: true }
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

            // Store completion timestamp for animation detection
            localStorage.setItem(`completion_time_${this.levelId}`, Date.now().toString())
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
        <div class="post-it-wrapper">
            <div class="post-it" :style="{ backgroundImage: `url(${postItUrl})` }">
            </div>
        </div>

        <!-- Notesbog with dynamic background -->
        <div class="notebook" :style="{ backgroundImage: `url(${notebookUrl})` }">
        </div>

        <!-- Spillets indhold -->
        <div class="game-content">
            <!-- Navigation buttons inside game content -->
            <div class="game-buttons">
                <button class="back-to-board-button" @click="goToBoard">
                   </button>
                        <button class="level-restart-button" @click="$emit('restart')">
                        </button>
            </div>

            <div class="game-main">
                <slot></slot>
            </div>

            <!-- Instruktioner -->
            <div class="instructions" :style="{ backgroundImage: `url(${instructionsUrl})` }">
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
    gap: var(--padding-medium);
    background-image: url('/images/ui/cork-board.png');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    padding: var(--padding-small);
    box-sizing: border-box;
    overflow: hidden;
}

/* Post-it */
.post-it-wrapper {
    background-size: contain;
    width: 30vh;
    height: 30vh;
    display: flex;
    align-items: center;
    justify-content: center;
    grid-column: 4 / 5;
    grid-row: 1 / 3;
    justify-self: center;
    align-self: start;
}

.post-it {
    width: 100%;
    height: 100%;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    position: relative;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    box-sizing: border-box;
}

/* Notesbog */
.notebook {
    grid-column: 4 / 5;
    grid-row: 3 / 6;
    padding-top: clamp(30px, 4vh, 60px);
    padding-left: clamp(20px, 3vw, 50px);
    width: clamp(350px, 30vh, 500px);
    height: 100%;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    display: flex;
    justify-content: center;
    overflow: hidden;
    justify-self: center;
    align-self: center;
}

/* Spilindhold */
.game-content {
    padding: var(--padding-small);
    grid-column: 1 / 4;
    grid-row: 1 / 7;
    background-color: #F1F4F0;
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
    top: var(--padding-small);
    left: var(--padding-small);
    right: var(--padding-small);
    display: flex;
    justify-content: space-between;
    z-index: 100;
}

/* Back to board button (top left) */
.back-to-board-button {
    background-image: url('/images/ui/back_btn.png');
    background-color: transparent;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    border: none;
    cursor: pointer;
    z-index: 1000;
    width: 30vh;
    height: 10vh;
}

.back-to-board-button:hover {
    transform: scale(1.05);
}

/* Level restart button (top right) */
.level-restart-button {
    background-image: url('/images/ui/startover_btn.png');
    background-color: transparent;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    border: none;
    cursor: pointer;
    z-index: 1000;
    width: 22vh;
    height: 10vh;
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
    padding-top: clamp(60px, 8vh, 100px);
}

/* Instruktionstekst */
.instructions {
    background-color: transparent;
    background-image: url('/images/ui/instructions_bg.png');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    padding: var(--padding-large);
    border: none;
    box-sizing: border-box;
    height: 15vh;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
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
    background-size: 100%;
    background-color: transparent;
    border: none;
    background-repeat: no-repeat;
    background-position: center;
    cursor: pointer;
    animation: pulse 2s ease-in-out infinite;
    background-image: url('/images/ui/continue_btn.png');
    width: 30vh;
    height: 10vh;
    z-index: 1000;
    justify-self: center;
    align-self: center;
}

.continue-button:hover {
    opacity: 1;
}
</style>