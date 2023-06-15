
<template>
  <div  style="width:100%;height:100%">
    <div id="demo"  style="width:100%;height:100%"></div>
  </div>
</template>
<script setup>
import { ref,onMounted } from 'vue'
import * as THREE from 'three'
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader';
import { Water } from "three/examples/jsm/objects/Water";
import { Sky } from 'three/addons/objects/Sky.js'; // 导入sky
import gsap from 'gsap'; // 导入补间动画工具
//导入gltf的载入库
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
// 因为我们的模型是压缩过的，所以需要导入解压的库
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';
defineProps({
  msg: String,
})

const count = ref(0)

onMounted(() => {
  
  // 初始化场景
  const scene = new THREE.Scene();
  let demo = document.getElementById('demo')
  let width = demo.clientWidth
  let height = demo.clientHeight
  // 初始化相机，使用透视相机
  const camera = new THREE.PerspectiveCamera(75,width/height,0.1,2000)
  // 设置相机的坐标
  camera.position.set(-50,50,130)

  // 设置相机的视锥的宽高比例
  camera.aspect = width/height
  // 更新相机的投影矩阵
  camera.updateProjectionMatrix()
  

  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer({
    antialias: true, // 抗锯齿
    logarithmicDepthBuffer: true, // 对数深度缓冲区，解决模块重叠以后的闪动问题
  })
  renderer.outputEncoding = THREE.sRGBEncoding; // 将渲染器的输出编码为sRGB
  renderer.setSize(width,height)
  const rendering = () => {
    water.material.uniforms[ 'time' ].value += 1.0 / 60.0;
    controls && controls.update();
    renderer.render(scene,camera)
    requestAnimationFrame(rendering)
  }

  //创建一个平面
  const circleBufferGeometry = new THREE.CircleBufferGeometry(10000,10000)
  const water = new Water(circleBufferGeometry,
    {
      textureWidth: 1024, //纹理宽度
      textureHeight: 1024, //纹理高度
      waterNormals: new THREE.TextureLoader().load( './img/islet/waternormals.jpg', function ( texture ) {
      texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
      } ),
      flowDirection: new THREE.Vector2(1, 1), // 流动方向
      scale: 1, // 水的缩放
      sunDirection: new THREE.Vector3(),
      sunColor: 0xffffff,
      waterColor: 0x0072ff,
      distortionScale: 4,
      fog: scene.fog !== undefined
    }
  )
  water.position.y = 3
  // 睡眠旋转90度
  water.rotation.x = - Math.PI * 0.5
  scene.add(water)

  // 添加小岛的模型 //
  const loader = new GLTFLoader()
  // 实例化解压库
  const dracoLoader = new DRACOLoader()
  // 添加解压库
  dracoLoader.setDecoderPath('/draco/')
  loader.setDRACOLoader(dracoLoader)
  // 加载小岛模型
  loader.load('./img/islet/model/island2.glb',(gltf)=>{
    //gltf.position.y= -11
    scene.add(gltf.scene) // 到这一步，因为环境没有灯光，所以模型是黑色的。有两个方案，第一增加灯光，2增加一个环境纹理,环境 纹理可以让整个环境都亮起来

  })
  //增加灯光
  const light = new THREE.DirectionalLight(0xffffff,1)
  light.position.set(-100,100,10)
  scene.add(light)
  // 增加环境纹理
  const hdrLoader = new RGBELoader()
  hdrLoader.loadAsync('./img/islet//050.hdr').then(texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    scene.background = texture
    scene.environment = texture
  })

  // 添加天空

  const sky = new Sky()
  sky.scale.setScalar(10000)
  scene.add(sky)
  // sky用的材质是ShaderMaterial（着色器材质），下面都是着色器等一些写法，可以不写
  let skyUniforms = sky.material.uniforms;
  skyUniforms['turbidity'].value = 1; // 浑浊度
  skyUniforms['rayleigh'].value = 3; // 视觉效果就是傍晚晚霞的红光的深度
  skyUniforms['mieCoefficient'].value = 0.005; //散射系数
  skyUniforms['mieDirectionalG'].value = 0.8; //定向散射值
  //luminance 视觉效果整体提亮或变暗
  // 太阳
  const sun = new THREE.Vector3();
  // 辐射贴图PMREMGenerator
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  const phi = THREE.MathUtils.degToRad(88);
  const theta = THREE.MathUtils.degToRad(180);
  sun.setFromSphericalCoords(1, phi, theta);
  sky.material.uniforms['sunPosition'].value.copy(sun);
  water.material.uniforms['sunDirection'].value.copy(sun).normalize();
  scene.environment = pmremGenerator.fromScene(sky).texture

 // 彩虹
 const material = new THREE.MeshNormalMaterial({
  transparent: true
 });
 material.opacity = 0.4;
  const geometry = new THREE.TorusGeometry(200, 10, 50, 100);
  const torus = new THREE.Mesh(geometry, material);
  
  torus.position.set(0, -50, -400);
  let position = torus.geometry.attributes.position
  console.log(torus)
  let torusCount = torus.geometry.attributes.position.count
  let c1 = new THREE.Color(0x0000ff) // 彩虹左侧底部颜色
  let c2 = new THREE.Color(0xff0000) // 彩虹右侧侧底部颜色
  let colorArr = []
  for (let i = 0; i < torusCount; i++) {
    let percent = (200 - position.getY(i)) / 200
    let c = c1.clone().lerp(c2,percent)
    colorArr.push(c.r,c.g,c.b)
  }
  const colors = new Float32Array(colorArr)
  console.log(geometry)
  geometry.attributes.color = new THREE.BufferAttribute(colors, 3)
  scene.add(torus);

//
  

  var controls = new OrbitControls(camera,demo)
  controls.target.set(0, 0, 0);
  controls.enableDamping = true;
  controls.enablePan = false;
  controls.maxPolarAngle = 1.5;
  controls.minDistance = 50;
  controls.maxDistance = 1200;
  // 在dom上渲染
  demo.appendChild(renderer.domElement)
  // 渲染
  rendering()
   
})

</script>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
