<template>
    <div class="file-container">
        <img src="/images/file.png" alt="" class="file-image" />

        <button class="start-button">
            <router-link to="/board">Start efterforskningen -></router-link>
        </button>

    </div>
</template>

<style scoped>
.file-container {
    font-family: 'Coming Soon', cursive;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    background-color: transparent;
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
    right: 12%;
    top: 77%;
    background-image: url('/images/button_long.png');
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