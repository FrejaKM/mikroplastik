<template>
    <div class="file-container">
        <div class="file-image-wrapper">
            <img src="/images/games+completion/file.png" alt="" class="file-image" />
            <button class="start-button">
                <router-link to="/board"></router-link>
            </button>
        </div>
    </div>
</template>

<style scoped>
.file-container {
    background-color: #F1F4F0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    box-sizing: border-box;
}

.file-image-wrapper {
    position: relative;
    display: inline-block;
}

.file-image {
    display: block;
    width: 140vh;
    max-width: 90vw;
    height: auto;
}

.start-button {
    position: absolute;
    border: none;
    background-image: url('/images/ui/begin_btn.png');
    background-color: transparent;
    background-position: center;
    background-repeat: no-repeat;
    z-index: 1000;
    cursor: pointer;
    animation: pulse 2s ease-in-out infinite;
    background-size: contain;
    right: 5%;
    bottom: 5%;
    width: 30vh;
    height: 10vh;
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