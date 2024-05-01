<template>
  <div class="hello">
    <div id="container" class="container"></div>
  </div>
</template>

<script>
import * as Three from 'three'

let scene = null,
    camera=null,
    renderer=null,
    mesh=null

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return{

    }
  },
  created(){
    console.log('第一次',Three.Scene);
  },
  methods:{

    init(){
      let container = document.getElementById('container');
      camera = new Three.PerspectiveCamera(70, container.clientWidth/container.clientHeight, 0.01, 10);
      camera.position.z = 1
      //创建3D场景对象Scene
      scene = new Three.Scene();
      //创建一个长方体几何对象Geometry
      let geometry = new Three.BoxGeometry(0.2, 0.5, 0.2);
      // let material = new Three.MeshNormalMaterial();
      //创建一个材质对象Material
      const material = new Three.MeshBasicMaterial({
        color: 0xff0000,//0xff0000设置材质颜色为红色
      });
      //两个参数分别为几何体geometry、材质material
      mesh = new Three.Mesh(geometry, material);
      //设置网格模型在三维空间中的位置坐标，默认是坐标原点
      // mesh.position.set(20,10,0);
      scene.add(mesh);

      renderer = new Three.WebGLRenderer({antialias:true});
      renderer.setSize(container.clientWidth,container.clientHeight);
      container.appendChild(renderer.domElement);

    },
    animate(){
      requestAnimationFrame(this.animate);
      console.log(this.animate,'132')
      mesh.rotation.x += 0.01;
      mesh.rotation.y += 0.02;
      renderer.render(scene,camera);
    }
  },
  mounted(){//页面渲染完毕之后，启用方法
    this.init()
    this.animate()

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#container{
  width: 100vw;
  height: 100vh;
}
</style>
