<template>
    <div class="file-container">
        <button>
            <router-link to="/board">Go to Board</router-link>
        </button>
    </div>
    <div class="logo-container">
        <img src="/images/logo.png" alt="Logo" class="logo" />
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

.logo-container {
    position: fixed;
    bottom: 6px;
    right: 8px;
    z-index: 1000;
}

.logo {
    width: 15vh;
    height: auto;
}
</style>