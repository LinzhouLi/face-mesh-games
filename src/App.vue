<script setup>

</script>

<template>
  <div id="bg" />
  <router-view />
</template>

<style>
.el-message-box {
  font-family: Comic Sans MS, Comic Sans, cursive;
}

#app {
  font-family: Comic Sans MS, Comic Sans, cursive;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#bg {
  background-image: url('@/assets/background.png');
  pointer-events: none;
  z-index: -1;
  position: fixed;
  top: 0;
  left: 50%;
  width: 100%;
  height: 100vh;
  transform: translateX(-50%);
  background-repeat: no-repeat;
  background-size: contain;
  background-position: top;
  background-color: #d1e9fb;
}
</style>
