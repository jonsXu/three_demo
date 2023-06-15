
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
    logarithmicDepthBuffer: true, // 对数深度缓冲区，解决一些阴影问题
  })
  renderer.outputEncoding = THREE.sRGBEncoding; // 将渲染器的输出编码为sRGB
  renderer.setSize(width,height)
  const rendering = () => {
    renderer.render(scene,camera)
    requestAnimationFrame(rendering)
  }

  let video = document.createElement('video')
  
  video.src = './img/islet/sky.mp4'
  video.autoplay = true // 自动播放
  video.loop = true //循环播放
  
  //加载天空纹理
  const texture = new THREE.TextureLoader().load('./img/islet/sky.jpg')

  // 实例化控制器
  // 创建一个球形几何体,纹理用天空
  const geometry = new THREE.SphereGeometry(1000,60,60)
  const mesh = new THREE.MeshBasicMaterial({
    map:texture ,//用图片天空纹理

  })
  // 因为浏览器默认会阻止视频自动播放，所以增加一个兼听事件
  window.addEventListener('click',()=>{
    if(video.paused){
      video.play()
      let texture = new THREE.VideoTexture(video)
      mesh.map = texture// 将video添加到纹理中
      mesh.map.needsUpdate = true
    }
  })
  const sky = new THREE.Mesh(geometry,mesh)
  sky.geometry.scale(1,1,-1)
  scene.add(sky)
  

  // 放一个湖水进来

  //创建一个平面
  const circleBufferGeometry = new THREE.CircleBufferGeometry(400,60)
  const water = new Water(circleBufferGeometry,
    {
      textureWidth: 1024, //纹理宽度
      textureHeight: 1024, //纹理高度
      color: 0xeeeeff, // 水的颜色
      flowDirection: new THREE.Vector2(1, 1), // 流动方向
      scale: 1, // 水的缩放
      sunDirection: new THREE.Vector3(),
      sunColor: 0xffffff,
      waterColor: 0x001e0f,
      distortionScale: 3.7,
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

  const control = new OrbitControls(camera,demo)
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
