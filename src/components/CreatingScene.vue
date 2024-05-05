<script>
/*
* To actually be able to display anything with three.js,
* we need three things: scene, camera and renderer, so that we can render the scene with camera.
* 为了真正能够让你的场景借助 three.js 来进行显示，
* 我们需要以下几个对象：场景（scene）、相机（camera）和渲染器（renderer），这样我们就能透过摄像机渲染出场景。
* */
import * as THREE from "three";
// import WebGL from 'three/examples/jsm/capabilities/WebGL.js'
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
  mounted(){//页面渲染完毕之后，启用方法
    this.init()
  },
  methods:{
    //Creating a scene
    init(){
      const scene = new THREE.Scene();
      //There are a few different cameras in three.js. For now, let's use a PerspectiveCamera.
      //three.js 里有几种不同的相机，在这里，我们使用的是 PerspectiveCamera（透视摄像机）。
      //第一个参数是视野角度（FOV）。视野角度就是无论在什么时候，你所能在显示器上看到的场景的范围，它的单位是角度(与弧度区分开)。
      //第二个参数是长宽比（aspect ratio）。 也就是你用一个物体的宽除以它的高的值。比如说，当你在一个宽屏电视上播放老电影时，可以看到图像仿佛是被压扁的。
      /*
      * 接下来的两个参数是近截面（near）和远截面（far）。
      * 当物体某些部分比摄像机的远截面远或者比近截面近的时候，该这些部分将不会被渲染到场景中。
      * 或许现在你不用担心这个值的影响，但未来为了获得更好的渲染性能，你将可以在你的应用程序里去设置它。
      * */
      const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

      const renderer = new THREE.WebGLRenderer();
      /*
      * 除了创建一个渲染器的实例之外，我们还需要在我们的应用程序里设置一个渲染器的尺寸。
      * 比如说，我们可以使用所需要的渲染区域的宽高，来让渲染器渲染出的场景填充满我们的应用程序。
      * 因此，我们可以将渲染器宽高设置为浏览器窗口宽高。
      * 对于性能比较敏感的应用程序来说，你可以使用 setSize 传入一个较小的值，例如 window.innerWidth/2 和 window.innerHeight/2，这将使得应用程序在渲染时，以一半的长宽尺寸渲染场景。
      * 如果你希望保持你的应用程序的尺寸，是以较低的分辨率来渲染，你可以在调用 setSize 时，将 updateStyle（第三个参数）设为 false。
      * 例如，假设你的 <canvas> 标签现在已经具有了 100% 的宽和高，调用 setSize(window.innerWidth/2, window.innerHeight/2, false) 将使得你的应用程序以四分之一的大小来进行渲染。
      * */
      renderer.setSize( window.innerWidth, window.innerHeight );
      document.body.appendChild( renderer.domElement );

      /*
      * 最后一步很重要，我们将 renderer（渲染器）的dom元素（renderer.domElement）添加到我们的 HTML 文档中。
      * 这就是渲染器用来显示场景给我们看的 <canvas> 元素。
      * */
      //To create a cube, we need a BoxGeometry. This is an object that contains all the points (vertices) and fill (faces) of the cube. We'll explore this more in the future.
      //要创建一个立方体，我们需要一个 BoxGeometry（立方体）对象. 这个对象包含了一个立方体中所有的顶点（vertices）和面（faces）。未来我们将在这方面进行更多的探索。
      const geometry = new THREE.BoxGeometry( 1, 1, 1 );
      //接下来，对于这个立方体，我们需要给它一个材质，来让它有颜色。
      //Three.js 自带了几种材质，在这里我们使用的是 MeshBasicMaterial。
      /*
      * 所有的材质都存有应用于他们的属性的对象。
      * 在这里为了简单起见，我们只设置一个color属性，值为 0x00ff00，也就是绿色。
      * 这里所做的事情，和在 CSS 或者 Photoshop 中使用十六进制（hex colors）颜色格式来设置颜色的方式一致。
      * */
      const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
      /*
      * 第三步，我们需要一个 Mesh（网格）。
      * 网格包含一个几何体以及作用在此几何体上的材质，我们可以直接将网格对象放入到我们的场景中，并让它在场景中自由移动。
      * */
      const cube = new THREE.Mesh( geometry, material );
      /*
      * 默认情况下，当我们调用 scene.add() 的时候，物体将会被添加到 (0,0,0) 坐标。
      * 但将使得摄像机和立方体彼此在一起。为了防止这种情况的发生，我们只需要将摄像机稍微向外移动一些即可。
      * */
      scene.add( cube );
      camera.position.z = 5;

      //渲染场景
      /*
      * 现在，如果将之前写好的代码复制到HTML文件中，你不会在页面中看到任何东西。
      * 这是因为我们还没有对它进行真正的渲染。
      * 为此，我们需要使用一个被叫做“渲染循环”（render loop）或者“动画循环”（animate loop）的东西。
      * */
      function animate() {
        requestAnimationFrame( animate );
        //使立方体动起来
        /*
        * 在开始之前，如果你已经将上面的代码写入到了你所创建的文件中，你可以看到一个绿色的方块。
        * 让我们来做一些更加有趣的事 —— 让它旋转起来。
        * 将下列代码添加到 animate() 函数中 renderer.render 调用的上方：
        * */
        cube.rotation.x += 0.01;
        cube.rotation.y += 0.01;
        renderer.render( scene, camera );
      }
      animate();

      //导入WebGL兼容检测模块，并在尝试渲染任何内容之前运行以下程序。
      // if ( WebGL.isWebGLAvailable() ) {
      //   // Initiate function or other initializations here
      //   animate();
      // } else {
      //   const warning = WebGL.getWebGLErrorMessage();
      //   document.getElementById( 'container' ).appendChild( warning );
      //   document.body.appendChild( renderer.domElement );
      // }

    },
  }
}
</script>

<template>
<div class="container">

</div>
</template>

<style scoped>

</style>
