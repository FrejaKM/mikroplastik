<script>
export default {
    name: "Base",
    props: {
        postItUrl: { type: String, required: true },
        title: { type: String, required: true },
        description: { type: String, required: true },
        instructions: { type: String, required: true },
        completed: { type: Boolean, default: false }
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
            <div class="info-text">{{ description }}</div>
        </div>

        <!-- Spillets indhold -->
        <div class="game-content">
            <slot></slot>
        </div>

        <!-- Instruktioner -->
        <div class="instructions">
            {{ instructions }}
        </div>

        <!-- Fortsæt-knap -->
        <button v-if="completed" class="continue-button" @click="$emit('continue')">
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
    /* Remove the background-color since we're using the cork board image */
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
    grid-row: 1 / 6;
    border: 2px solid #000000;
    background-color: #eef5ef;
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
}

/* Instruktionstekst */
.instructions {
    grid-column: 1 / 4;
    grid-row: 6 / 7;
    background-color: none;
    border: 2px solid black;
    background-color: #eef5ef;
    border-radius: 10px;
    font-size: 32px;
    padding: 10px 14px;
    display: flex;
    align-items: center;
    overflow: hidden;
    box-sizing: border-box;
    min-height: 0;
    min-width: 0;
}

/* Fortsæt-knap */
.continue-button {
    grid-column: 4 / 5;
    grid-row: 6 / 7;
    border: 2px solid black;
    border-radius: 10px;
    background-color: #eef5ef;
    cursor: pointer;
    font-size: 32px;
    padding: 8px 18px;
}

.continue-button:hover {
    scale: 1.02;
}
</style>