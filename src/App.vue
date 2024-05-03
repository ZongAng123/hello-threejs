<template>
<!--  <img alt="Vue logo" src="./assets/logo.png">-->
  <HelloWorld />
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
// import HelloWorld from './components/CreatingScene.vue' //创建场景
// import HelloWorld from './components/WebGLCompatibility.vue'
// import HelloWorld from './components/Drawinglines.vue' //画线
import HelloWorld from './components/Creatingtext.vue' //创建文字

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: -10px;
}
</style>
