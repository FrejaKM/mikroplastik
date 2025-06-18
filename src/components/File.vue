<template>
    <div class="file-container">
        <img src="/images/games+completion/file.png" alt="" class="file-image" />

        <button class="start-button">
            <router-link to="/board">Start efterforskningen -></router-link>
        </button>

    </div>
</template>

<style scoped>
.file-container {
    background-color: #F1F4F0;
    font-family: 'Coming Soon', cursive;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: top;
    width: 100vw;
    height: 100vh;
    padding-top: var(--padding-small);
    box-sizing: border-box;
}

.file-image {
    display: block;
    width: clamp(300px, 80vw, 1200px);
    height: auto;
}

.start-button {
    position: absolute;
    border: 1px solid #000000;
    padding: var(--padding-small) var(--padding-medium);
    text-align: center;
    text-decoration: none;
    display: inline-block;
    right: var(--padding-large);
    bottom: clamp(20px, 2vh, 40px);
    background-image: url('/images/ui/button_long.png');
    background-color: transparent;
    width: auto;
    height: var(--button-height);
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    color: rgb(0, 0, 0);
    padding: var(--padding-medium);
    font-size: var(--font-medium);
    cursor: pointer;
    z-index: 1000;
    border: none;
    animation: pulse 2s ease-in-out infinite;
}

.start-button:hover {
    transform: scale(1.05);
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
</style>