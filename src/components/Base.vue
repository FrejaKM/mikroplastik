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
                <div class="post-it-text">{{ title }}</div>
            </div>
        </div>

        <!-- Notesbog -->
        <div class="notebook" style="background-image: url('/images/ui/notebook_page.png')">
            <div class="description" v-html="description"></div>
        </div>

        <!-- Spillets indhold -->
        <div class="game-content">
            <!-- Navigation buttons inside game content -->
            <div class="game-buttons">
                <button class="back-to-board-button" @click="goToBoard">
                    <- Tilbage til opslagstavlen </button>
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
        <button v-if="completed" class="continue-button" @click="handleContinue">Fortsæt -></button>
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
    background-image: url('/images/ui/cork-board.png');
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    padding: 20px;
    box-sizing: border-box;
    overflow: hidden;
}

/* Post-it */
.post-it-wrapper {
    background-size: contain;
    position: absolute;
    width: 250px;
    /* Fixed width */
    height: 250px;
    /* Adjust as needed for your post-it image */
    z-index: 100;
    top: 30px;
    right: 75px;
    display: flex;
    align-items: center;
    justify-content: center;
    /* Remove 5vw height, use px for consistency */
}

.post-it {
    width: 100%;
    /* Fixed width */
    height: 100%;
    /* Match wrapper height */
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

.post-it-text {
    color: #000000;
    font-weight: 400;
    text-align: center;
    font-family: 'Coming Soon', cursive;
    font-size: 1.6vw;
    text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
    z-index: 2;
    padding: 5px;
}


/* Notesbog */
.notebook {
    grid-column: 4 / 5;
    grid-row: 3 / 6;
    padding-top: 50px;
    padding-left: 40px;
    width: 300px;
    height: 400px;
    background-size: 320px 420px;
    background-repeat: no-repeat;
    background-position: center;
    display: flex;
    justify-content: center;
    overflow: hidden;
    justify-self: center;
    /* Center horizontally in grid cell */
    align-self: center;
    /* Center vertically in grid cell (optional) */
}

.description {
    padding-top: 10px;
    width: 230px;
    height: 100%;
    font-size: 20px;
    font-family: 'Coming Soon', cursive;
    text-align: left;
    font-weight: 300;
    color: black;
    box-sizing: border-box;
    line-height: 1.5;
}

/* Spilindhold */
.game-content {
    padding: 20px;
    grid-column: 1 / 4;
    grid-row: 1 / 7;
    background-color: #F1F4F0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
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
    background-image: url('/images/ui/button_long.png');
    top: 20px;
    background-color: transparent;
    width: 290px;
    background-size: 100%;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    background-repeat: no-repeat;
    color: rgb(0, 0, 0);
    padding: 20px 20px;
    font-size: 20px;
    cursor: pointer;
    z-index: 1000;
    border: none;
}

.back-to-board-button:hover {
    transform: scale(1.05);
}

/* Level restart button (top right) */
.level-restart-button {
    background-image: url('/images/ui/button_short.png');
    top: 20px;
    background-color: transparent;
    left: 15px;
    width: 240px;
    background-size: 100%;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    background-repeat: no-repeat;
    color: rgb(0, 0, 0);
    padding: 20px 20px;
    font-size: 20px;
    cursor: pointer;
    z-index: 1000;
    border: none;
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
    background-color: transparent;
    background-image: url('/images/ui/instructions_bg.png');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    color: rgb(0, 0, 0);
    padding: 40px 40px;
    font-size: 25px;
    border: none;
    line-height: 1.5;
    text-align: center;
    box-sizing: border-box;
    width: 100%;
    max-width: 1050px;
    min-height: 130px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 0 auto;
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

    background-image: url('/images/ui/button_short.png');
    width: 280px;
    height: 100px;
    font-family: 'Coming Soon', cursive;
    background-repeat: no-repeat;
    color: rgb(0, 0, 0);
    padding: 20px 20px;
    padding-top: 10px;
    font-size: 20px;
    cursor: pointer;
    z-index: 1000;
    border: none;
    justify-self: center;
    /* Center horizontally in grid cell */
    align-self: center;
    /* Center vertically in grid cell (optional) */
}

.continue-button:hover {
    opacity: 1;
}
</style>