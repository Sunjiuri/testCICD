<template>
  <div id="app">
    <header class="header">
      <h1>Vue+Three.js 3D效果</h1>
      <p>使用鼠标拖动旋转，滚轮缩放</p>
    </header>
    <ThreeScene ref="threeScene" />

    <div class="controls">
      <button @click="toggleEffect">切换效果</button>
      <button @click="changeColor">改变颜色</button>
    </div>
  </div>
</template>

<script>
import ThreeScene from "./components/ThreeScene.vue";

export default {
  name: "App",
  components: {
    ThreeScene
  },
  methods: {
    toggleEffect() {
      if (this.$refs.threeScene) {
        this.$refs.threeScene.toggleEffect();
      }
    },
    changeColor() {
      if (this.$refs.threeScene) {
        this.$refs.threeScene.changeColor();
      }
    }
  }
};
</script>

<style>
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app{
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.header{
  text-align: center;
  padding: 20px;
  background: rgba(255,255,255,0.9);
  backdrop-filter: blur(10px);
  position: relative;
  z-index: 100;
}

.controls{
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
}

.controls button{
  margin: 0 10px;
  padding: 10px 20px;
  background:#4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}
</style>