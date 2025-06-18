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
    justify-content: center;
    width: 100vw;
    height: 100vh;
    padding: 20px;
    box-sizing: border-box;
}

.file-image {
    display: block;
    width: 85vw;
    height: auto;
    margin-bottom: 20px;
}


.start-button {
    position: absolute;
    border: 1px solid #000000;
    padding: 10px 20px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    right: 12vw;
    top: 80vh;
    background-image: url('/images/ui/button_long.png');
    background-color: transparent;
    width: 15vw;
    height: 10vh;
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
    font-family: 'Coming Soon', cursive;
    font-weight: 300;
    color: rgb(0, 0, 0);
    padding: 20px 20px;
    font-size: 20px;
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