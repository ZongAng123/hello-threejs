<template>
  <div class="hello">
    <div id="webgl" class="webgl"></div>
  </div>
</template>

<script>
import * as Three from 'three'
// 引入轨道控制器扩展库OrbitControls.js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';


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


      /*
      * 4、第一个3D案例—创建3D场景
      * */
      //创建3D场景对象Scene
      const scene = new Three.Scene();
      //创建一个长方体几何对象Geometry
      //创建盒子几何体，设置宽、高、深度
      // const geometry = new Three.BoxGeometry();
      const geometry = new Three.BoxGeometry(0.3, 0.3, 0.3);
      //创建一个材质对象Material
      //MeshBasicMaterial不受光照影响
      const material = new Three.MeshBasicMaterial();
      //MeshLambertMaterial受光照影响
      // const material = new Three.MeshLambertMaterial();
      //材质半透明设置
      // const material  = new Three.MeshBasicMaterial({
      //   color: 0x0000ff, //设置材质颜色
      //   transparent:true,//开启透明
      //   opacity:0.5,//设置透明度
      // });
      //物体：网格模型Mesh
      // 两个参数分别为几何体geometry、材质material
      const mesh = new Three.Mesh(geometry, material); //网格模型对象Mesh
      //设置网格模型在三维空间中的位置坐标，默认是坐标原点
      // mesh.position.set(100,0,0);
      //在threejs中你创建了一个表示物体的虚拟对象Mesh，需要通过.add()方法，把网格模型mesh添加到三维场景scene中。
      scene.add(mesh);


      /*
      * 5、第一个3D案例—透视投影相机
      * Threejs如果想把三维场景Scene渲染到web网页上，还需要定义一个虚拟相机Camera，
      * 就像你生活中想获得一张照片，需要一台用来拍照的相机。
      * */
      // 实例化一个透视投影相机对象
      let container = document.getElementById('webgl');
      const camera = new Three.PerspectiveCamera(70, container.clientWidth/container.clientHeight, 0.1, 10)
      camera.position.z = 1
      //相机在Three.js三维坐标系中的位置
      // 根据需要设置相机位置具体值
      // camera.position.set(20, 20, 20);
      //相机观察目标指向Threejs 3D空间中某个位置
      // camera.lookAt(mesh.position);//指向mesh对应的位置
      // width和height用来设置Three.js输出的Canvas画布尺寸(像素px)
      // 30:视场角度, width / height:Canvas画布宽高比, 1:近裁截面, 3000：远裁截面
      // PerspectiveCamera( fov, aspect, near, far )
      // const camera = new Three.PerspectiveCamera(30, width / height, 1, 3000);

    /*
    * 6. 第一个3D案例—渲染器
    * WebGL渲染器WebGLRenderer
    * */
      // 创建渲染器对象
      const renderer = new Three.WebGLRenderer({antialias:true});
      renderer.setSize(container.clientWidth,container.clientHeight);
      //渲染器渲染方法.render()
      renderer.render(scene, camera); //执行渲染操作
      // 将渲染器的DOM元素添加到HTML容器中
      // container.appendChild(renderer.domElement);
      document.getElementById('webgl').appendChild(renderer.domElement);
      //渲染器Canvas画布属性.domElement
      // document.body.appendChild(renderer.domElement);


      // 辅助观察坐标系
      // Three.AxesHelper()的参数表示坐标系坐标轴线段尺寸大小，你可以根据需要改变尺寸。
      // AxesHelper：辅助观察的坐标系
      const axesHelper = new Three.AxesHelper(150);
      scene.add(axesHelper);

      //点光源：两个参数分别表示光源颜色和光照强度
      // 参数1：0xffffff是纯白光,表示光源颜色
      // 参数2：1.0,表示光照强度，可以根据需要调整
      const pointLight = new Three.PointLight(0xffffff, 1.0);
      pointLight.intensity = 1.0;//光照强度
      pointLight.decay = 0.0;//设置光源不随距离衰减
      //点光源位置
      pointLight.position.set(400, 0, 0);//点光源放在x轴上
      scene.add(pointLight); //点光源添加到场景中


      /*
      * 9. 相机控件OrbitControls
      * 平时开发调试代码，或者展示模型的时候，可以通过相机控件OrbitControls实现旋转缩放预览效果。
      * */
      // 设置相机控件轨道控制器OrbitControls
      const controls = new OrbitControls(camera, renderer.domElement);
      // 如果OrbitControls改变了相机参数，重新调用渲染器渲染三维场景
      controls.addEventListener('change', function () {
        renderer.render(scene, camera); //执行渲染操作
        // 浏览器控制台查看相机位置变化
        console.log('camera.position',camera.position);
      });//监听鼠标、键盘事件

    /*
    * 10. 平行光与环境光
    * 点光源辅助观察PointLightHelper
    * */
      // 光源辅助观察
      const pointLightHelper = new Three.PointLightHelper(pointLight, 10);
      scene.add(pointLightHelper);
      //改变点光源位置，观察光照效果变化。
      pointLight.position.set(100, 60, 50);
     // 改变点光源位置，使用OrbitControls辅助观察
      pointLight.position.set(-400, -200, -300);

      //环境光设置
      //环境光:没有特定方向，整体改变场景的光照明暗
      const ambient = new Three.AmbientLight(0xffffff, 0.4);
      scene.add(ambient);

      /*
      * 平行光
      * 平行光DirectionalLight (opens new window)就是沿着特定方向发射。
      * */
      // 平行光
      const directionalLight = new Three.DirectionalLight(0xffffff, 1);
      // 设置光源的方向：通过光源position属性和目标指向对象的position属性计算
      directionalLight.position.set(80, 100, 50);
      // 方向光指向对象网格模型mesh，可以不设置，默认的位置是0,0,0
      directionalLight.target = mesh;
      scene.add(directionalLight);

      //平行光辅助观察DirectionalLightHelper
      // DirectionalLightHelper：可视化平行光
      const dirLightHelper = new Three.DirectionalLightHelper(directionalLight, 5,0xff0000);
      scene.add(dirLightHelper);
      // 对比不同入射角，mesh表面对光照的反射效果
      directionalLight.position.set(100, 0, 0);
      directionalLight.position.set(0, 100, 0);
      directionalLight.position.set(100, 100, 100);
      directionalLight.position.set(100, 60, 50);
      //directionalLight.target默认指向坐标原点

     /*
     * 11. 动画渲染循环
     * threejs可以借助HTML5的API请求动画帧window.requestAnimationFrame实现动画渲染。
     * 请求动画帧window.requestAnimationFrame
     * */

      // 渲染场景
      // const animate = () => {
      //   requestAnimationFrame(animate);
      //   mesh.rotation.x += 0.01;
      //   mesh.rotation.y += 0.01;
      //   renderer.render(scene, camera);
      // };
      // animate();

      // 计算两帧渲染时间间隔和帧率
      const clock = new Three.Clock();
      function render() {
        const spt = clock.getDelta()*1000;//毫秒
        console.log('两帧渲染时间间隔(毫秒)',spt);
        console.log('帧率FPS',1000/spt);
        renderer.render(scene, camera); //执行渲染操作
        mesh.rotateY(0.01);//每次绕y轴旋转0.01弧度
        requestAnimationFrame(render);//请求再次执行渲染函数render，渲染下一帧
      }
      render();


    },

  },
  mounted(){//页面渲染完毕之后，启用方法
    this.init()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#webgl{
  width: 100vw;
  height: 100vh;
}
</style>
