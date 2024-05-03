<script >
import * as THREE from "three";
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js';
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js';
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js';

//Cache:一个简单的缓存系统，内部使用FileLoader。
//要在所有使用FileLoader的加载器上启用缓存， 需设置
THREE.Cache.enabled = true;//是否启用缓存，默认为false.

let container;
let camera, cameraTarget, scene, renderer;
let group, textMesh1, textMesh2, textGeo, materials;
let firstLetter = true;

let text = 'China',
    bevelEnabled = true,
    font = undefined,
    fontWeight = 'bold'; // normal bold

const depth = 20,
    size = 70,
    hover = 30,
    curveSegments = 4,
    bevelThickness = 2,
    bevelSize = 1.5;

const mirror = true;
const fontMap = {
  'helvetiker': 0,
  'optimer': 1,
  'gentilis': 2,
  'droid/droid_sans': 3,
  'droid/droid_serif': 4
};
const weightMap = {
  'regular': 0,
  'bold': 1
};
const reverseFontMap = [];
const reverseWeightMap = [];

for ( const i in fontMap ) reverseFontMap[ fontMap[ i ] ] = i;
for ( const i in weightMap ) reverseWeightMap[ weightMap[ i ] ] = i;

let targetRotation = 0;
let targetRotationOnPointerDown = 0;

let pointerX = 0;
let pointerXOnPointerDown = 0;

let windowHalfX = window.innerWidth / 2;
// let fontIndex = 1;

init();
animate();

function init(){
  container = document.createElement( 'div' );
  document.body.appendChild( container );

  //透视相机
  /*
  * fov — 摄像机视锥体垂直视野角度
  * aspect — 摄像机视锥体长宽比
  * near — 摄像机视锥体近端面
  * far — 摄像机视锥体远端面
  * */
  camera = new THREE.PerspectiveCamera( 30, window.innerWidth / window.innerHeight, 1, 1500 );
  camera.position.set( 0, 400, 700 );
  //三维向量(Vector3) Vector3( x : Float, y : Float, z : Float )
  /*
  * 该类表示的是一个三维向量（3D vector）。
  *  一个三维向量表示的是一个有顺序的、三个为一组的数字组合（标记为x、y和z）， 可被用来表示很多事物.
  * 例如：
  * 一个位于三维空间中的点。
  * 一个在三维空间中的方向与长度的定义。
  * 在three.js中，长度总是从(0, 0, 0)到(x, y, z)的 Euclidean distance(欧几里德距离，即直线距离),方向也是从(0, 0, 0)到(x, y, z)的方向。
  * 任意的、有顺序的、三个为一组的数字组合。
  * */
  cameraTarget = new THREE.Vector3( 0, 150, 0 );

  // 创建3D场景
  scene = new THREE.Scene();
  //设置背景色
  scene.background = new THREE.Color( 0x000000 );
  /*
  * 雾(Fog):这个类中的参数定义了线性雾。也就是说，雾的密度是随着距离线性增大的。
  * 构造器:Fog( color : Integer, near : Float, far : Float )
  * */
  scene.fog = new THREE.Fog( 0x000000, 250, 1400 );

  // 平行光(DirectionalLight)
  // 平行光是沿着特定方向发射的光。这种光的表现像是无限远，从它发出的光线都是平行的。常常用平行光来模拟太阳光的效果。
  /*
  * DirectionalLight( color : Color, intensity : Float )
  * color -（可选）一个表示颜色的 Color 的实例、字符串或数字，默认为一个白色（0xffffff）的 Color 对象。
  * intensity -（可选）光照的强度。默认值为 1。
  * */
  const dirLight = new THREE.DirectionalLight( 0xffffff, 0.4 );
  dirLight.position.set( 0, 0, 1 ).normalize();
  scene.add( dirLight );

  //点光源（PointLight）
  /*
  * 从一个点向各个方向发射的光源。一个常见的例子是模拟一个灯泡发出的光。
  * PointLight( color : Color, intensity : Float, distance : Number, decay : Float )
  * color -（可选）一个表示颜色的 Color 的实例、字符串或数字，默认为一个白色（0xffffff）的 Color 对象。
  * intensity -（可选）光照强度。默认值为 1。
  * distance - 光源照射的最大距离。默认值为 0（无限远）。
  * decay - 沿着光照距离的衰退量。默认值为 2。
  * */
  const pointLight = new THREE.PointLight( 0xffffff, 4.5, 0, 0 );
  //随机生成当前每个粒子的亮度
  pointLight.color.setHSL( Math.random(), 1, 0.5 );
  pointLight.position.set( 0, 100, 90 );
  scene.add( pointLight );

  materials = [
    //MeshPhongMaterial(网格材质，一种用于具有镜面高光的光泽表面的材质。)
    /*
    * MeshPhongMaterial( parameters : Object )
    * parameters - (可选)用于定义材质外观的对象，具有一个或多个属性。 材质的任何属性都可以从此处传入(包括从Material继承的任何属性)。
    * 属性color例外，其可以作为十六进制字符串传递，默认情况下为 0xffffff（白色），内部调用Color.set(color)。
    * */
    new THREE.MeshPhongMaterial( { color: 0xffffff, flatShading: true } ), // front
    new THREE.MeshPhongMaterial( { color: 0xffffff } ) // side
  ];

  //Group(‘组’的意思)
  /*
  * 它几乎和Object3D是相同的，其目的是使得组中对象在语法上的结构更加清晰。
  * */
  group = new THREE.Group();
  group.position.y = 100;
  scene.add( group );

  loadFont();

  //创建网格 Mesh
  const plane = new THREE.Mesh(
      //平面缓冲几何体
      new THREE.PlaneGeometry( 10000, 10000 ),
      //基础网格材质(一个以简单着色（平面或线框）方式来绘制几何体的材质。)
      /*
      * 这种材质不受光照的影响。
      * MeshBasicMaterial( parameters : Object )
      * parameters - (可选)用于定义材质外观的对象，具有一个或多个属性。材质的任何属性都可以从此处传入(包括从Material继承的任何属性)。
      * */
      new THREE.MeshBasicMaterial( { color: 0xffffff, opacity: 0.5, transparent: true } )
  );
  plane.position.y = 100;
  plane.rotation.x = - Math.PI / 2;
  scene.add( plane );

  //渲染器
  renderer = new THREE.WebGLRenderer( { antialias: true } );
  //设置设备像素比。通常用于避免HiDPI设备上绘图模糊
  renderer.setPixelRatio( window.devicePixelRatio );
  //将输出canvas的大小调整为(width, height)并考虑设备像素比，且将视口从(0, 0)开始调整到适合大小 将updateStyle设置为false以阻止对canvas的样式做任何改变。
  renderer.setSize( window.innerWidth, window.innerHeight );
  //渲染到DOM上
  container.appendChild( renderer.domElement );

  //EVENTS
  container.style.touchAction = 'none';
  container.addEventListener( 'pointerdown', onPointerDown );
  document.addEventListener( 'keypress', onDocumentKeyPress );
  document.addEventListener( 'keydown', onDocumentKeyDown );

  const params = {
    changeColor: function () {

      pointLight.color.setHSL( Math.random(), 1, 0.5 );

    },
    changeFont: function () {
      loadFont();
    },
    changeWeight: function () {
      if ( fontWeight === 'bold' ) {
        fontWeight = 'regular';
      } else {
        fontWeight = 'bold';
      }
      loadFont();
    },
    changeBevel: function () {
      bevelEnabled = ! bevelEnabled;
      refreshText();
    }
  };
  const gui = new GUI();
  gui.add( params, 'changeColor' ).name( 'change color' );
  gui.add( params, 'changeFont' ).name( 'change font' );
  gui.add( params, 'changeWeight' ).name( 'change weight' );
  gui.add( params, 'changeBevel' ).name( 'change bevel' );
  gui.open();
  window.addEventListener( 'resize', onWindowResize );
}

function onWindowResize() {
  windowHalfX = window.innerWidth / 2;
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize( window.innerWidth, window.innerHeight );
}

function onDocumentKeyDown( event ) {
  if ( firstLetter ) {
    firstLetter = false;
    text = '';
  }

  const keyCode = event.keyCode;
  // backspace
  if ( keyCode == 8 ) {
    event.preventDefault();
    text = text.substring( 0, text.length - 1 );
    refreshText();
    return false;
  }

}

function onDocumentKeyPress( event ) {
  const keyCode = event.which;
  // backspace
  if ( keyCode == 8 ) {
    event.preventDefault();
  } else {

    const ch = String.fromCharCode( keyCode );
    text += ch;
    refreshText();
  }
}

function loadFont() {
  //创建加载器
  /*
  * 一个用于加载JSON格式的字体的类。
  * 返回font, 返回值是表示字体的Shape类型的数组。其内部使用FileLoader来加载文件。
  * 你可以使用facetype.js来在线转换字体。
  * .load ( url : String, onLoad : Function, onProgress : Function, onError : Function ) : undefined
  * url — 文件的URL或者路径，也可以为 Data URI。
  * onLoad — 将在加载完成时调用。参数是将要被加载的font。
  * onProgress — 将在加载过程中调用。参数是包含total和loaded字节的XMLHttpRequest实例。
  * onError — 将在加载错误时调用。
  * 开始加载url，并将加载的font传递给onLoad。
  * */
  const loader = new FontLoader();
  //https://threejs.org/examples/fonts/optimer_bold.typeface.json
  //https://threejs.org/examples/fonts/helvetiker_regular.typeface.json
  //https://threejs.org/examples/fonts/helvetiker_bold.typeface.json
  //https://threejs.org/examples/fonts/helvetiker_regular.typeface.json
  //https://threejs.org/examples/fonts/optimer_regular.typeface.json
  //https://threejs.org/examples/fonts/gentilis_regular.typeface.json
  //https://threejs.org/examples/fonts/gentilis_bold.typeface.json
  //https://threejs.org/examples/fonts/droid/droid_sans_regular.typeface.json
  //https://threejs.org/examples/fonts/droid/droid_sans_bold.typeface.json 没显示
  //https://threejs.org/examples/fonts/droid/droid_serif_regular.typeface.json
  //https://threejs.org/examples/fonts/droid/droid_serif_bold.typeface.json
  loader.load( 'https://threejs.org/examples/fonts/droid/droid_serif_bold.typeface.json', function ( response ) {
    font = response;
    refreshText();
  } );
}

function refreshText() {
  group.remove( textMesh1 );
  if ( mirror ) group.remove( textMesh2 );
  if ( ! text ) return;
  createText();

}

function createText() {

  textGeo = new TextGeometry( text, {
    font: font,
    size: size,
    depth: depth,
    curveSegments: curveSegments,
    bevelThickness: bevelThickness,
    bevelSize: bevelSize,
    bevelEnabled: bevelEnabled
  });

  textGeo.computeBoundingBox();
  const centerOffset = - 0.5 * ( textGeo.boundingBox.max.x - textGeo.boundingBox.min.x );
  textMesh1 = new THREE.Mesh( textGeo, materials );
  textMesh1.position.x = centerOffset;
  textMesh1.position.y = hover;
  textMesh1.position.z = 0;
  textMesh1.rotation.x = 0;
  textMesh1.rotation.y = Math.PI * 2;
  group.add( textMesh1 );
  if ( mirror ) {
    textMesh2 = new THREE.Mesh( textGeo, materials );
    textMesh2.position.x = centerOffset;
    textMesh2.position.y = - hover;
    textMesh2.position.z = depth;
    textMesh2.rotation.x = Math.PI;
    textMesh2.rotation.y = Math.PI * 2;
    group.add( textMesh2 );
  }
}

function onPointerDown( event ) {

  if ( event.isPrimary === false ) return;
  pointerXOnPointerDown = event.clientX - windowHalfX;
  targetRotationOnPointerDown = targetRotation;
  document.addEventListener( 'pointermove', onPointerMove );
  document.addEventListener( 'pointerup', onPointerUp );

}

function onPointerMove( event ) {
  if ( event.isPrimary === false ) return;
  pointerX = event.clientX - windowHalfX;
  targetRotation = targetRotationOnPointerDown + ( pointerX - pointerXOnPointerDown ) * 0.02;
}

function onPointerUp(event) {
  if ( event.isPrimary === false ) return;
  document.removeEventListener( 'pointermove', onPointerMove );
  document.removeEventListener( 'pointerup', onPointerUp );

}

function animate() {
  requestAnimationFrame( animate );
  render();
}

function render() {
  group.rotation.y += ( targetRotation - group.rotation.y ) * 0.05;
  camera.lookAt( cameraTarget );
  renderer.clear();
  renderer.render( scene, camera );

}

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Creatingtext',
  props:{
  },
  data(){
    return{

    }
  },
  created() {
    this.init();
    this.animate();
  },
  mounted() {

  },
  methods:{

    init(){

    },
    animate(){

    }
  }
}
</script>

<template>
  <div id="info">
  </div>
</template>

<style scoped>
#container{
  width: 100vw;
  height: 100vh;
}
body {
  background-color: #bfe3dd;
  color: #000;
}

a {
  color: #2983ff;
}

</style>
