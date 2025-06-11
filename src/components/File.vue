<template>
    <div class="file-container">
        <img src="/images/file.png" alt="" class="file-image" />
        <div class="file-text">
            <h1>Du har fundet en fil, der indeholder vigtige oplysninger om efterforskningen.</h1>
            <p>Læs den grundigt for at finde spor, der kan hjælpe dig med at opklare sagen.</p>
        </div>
        <button>
            <router-link to="/board">Start!</router-link>
        </button>
        <div class="logo-container">
            <img src="/images/logo.png" alt="Logo" class="logo" />
        </div>
    </div>
</template>

<style scoped>
.file-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    background-color: #eef5ef;
    padding: 20px;
    box-sizing: border-box;
}

.file-image {
    width: 90vw;
    height: auto;
    margin-bottom: 20px;
}


.file-text {
    position: absolute;
    text-align: left;
    left: 55%;
    top: 10%;
    max-width: 500px;
    margin-bottom: 20px;
}

button {
    position: absolute;
    color: white;
    border: 1px solid #000000;
    padding: 10px 20px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    right: 10%;
    top: 83%;
    font-size: 16px;
    margin-top: 20px;
    cursor: pointer;
}

.logo-container {
    position: fixed;
    bottom: 15%;
    left: 20%;
    z-index: 1000;
}

.logo {
    width: 25vh;
    height: auto;
}
</style>