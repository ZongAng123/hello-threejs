<script>
/*
* 请将https://github.com/mrdoob/three.js/blob/master/examples/jsm/capabilities/WebGL.js引入到你的文件，
* 并在尝试开始渲染之前先运行该文件。
* */
import WebGL from 'three/examples/jsm/capabilities/WebGL.js'
export default{
  name: 'CreatingScene',
  props: {
    msg: String
  },
  data(){
    return{

    }
  },
  created(){

  },
  mounted(){
    this.init()
  },
  methods:{
    //Creating a scene
    init(){
      if ( WebGL.isWebGLAvailable() ) {

        // Initiate function or other initializations here
        // animate();

      } else {

        const warning = WebGL.getWebGLErrorMessage();
        document.getElementById( 'container' ).appendChild( warning );

      }
    },
  }
}
</script>

<template>
  <div>

  </div>
</template>

<style scoped>

</style>

<!--
WebGL compatibility check[WebGL兼容性检查]



-->
