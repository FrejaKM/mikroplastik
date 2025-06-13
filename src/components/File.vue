<template>
    <div class="file-container">
        <img src="/images/file.png" alt="" class="file-image" />
        <div class="file-text">
            <p class="header">EFTERFORSKNING <br> MILJØSAG #001: MIKROPLAST</p>
            <p>Et mystisk fund er gjort på en fiskeauktion i Hirtshals: En fiskemave er blevet undersøgt, og inde i den
                fandt man noget, der hverken hører til i havet eller i dets beboere – plastik! <br> <br> Hvordan er det
                havnet der? Og hvad betyder det for havets dyr og os mennesker?
                <br> <br> Det er tid til at opklare sagen, og som
                detektiv er det nu din opgave at følge sporene og opklare mysteriet
                om, hvordan mikroplasten sneg sig ud i havet.
            </p>
        </div>
        <button>
            <router-link to="/board">Start!</router-link>
        </button>
        <div class="logo-container">
            <img src="/images/logo_black.png" alt="Logo" class="logo" />
        </div>
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
    font-size: 24px;
    position: absolute;
    text-align: left;
    left: 55%;
    top: 10%;
    max-width: 500px;
    margin-bottom: 20px;
}

.header {
    font-size: 30px;
    font-weight: bold;
    margin-bottom: 10px;
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
    position: relative;
    bottom: 100px;
    right: 350px;
    z-index: 1000;
}

.logo {
    width: 15vw;
    height: auto;
}
</style>