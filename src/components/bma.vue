
<template>
  <div  style="width:100%;height:100%; position:relative">
    <div id="demo"  style="width:100%;height:100%"></div>
    <div class="tools">
      <div class="carbody">
        <div class="tabs">
          <div class="tab" @click="changeTab(1)">车轮</div>
          <div class="tab" @click="changeTab(2)">车身</div>
          <div class="tab" @click="changeTab(3)">车脸</div>
          <div class="tab" @click="changeTab(4)">引擎盖</div>
        </div>
        <div class="colors">
          <div class="coloritem" @click="changeColor(item)" v-for="item in colors" :key="item"
           :style="{background: item}">

          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref,onMounted } from 'vue'
import * as THREE from 'three'
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
//导入gltf的载入库
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
// 因为我们的模型是压缩过的，所以需要导入解压的库
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';
defineProps({
  msg: String,
})
let carBody = null // 车身
let fontCar = null // 车脸
let hoodCar = null // 前引擎盖
let wheels = [] //车轮，有四个
let nowTab = 2
let wheelsMaterial = new THREE.MeshPhysicalMaterial({
  color: 0X9B30FF,
  metalness:1, // 金属度
  roughness:0.5, // 粗糙程度
})
let carBodyMaterial = new THREE.MeshPhysicalMaterial({
  color: 0Xff0000,
  metalness:1, // 金属度
  roughness:0.5, // 粗糙程度
  clearcoat:1,
  clearcoatRoughness:1
})
let fontCarMaterial = new THREE.MeshPhysicalMaterial({
  color: 0Xff0000,
  metalness:1, // 金属度
  roughness:0.5, // 粗糙程度
  clearcoat:1,
  clearcoatRoughness:1
})
let hoodCarMaterial = new THREE.MeshPhysicalMaterial({
  color: 0Xff0000,
  metalness:1, // 金属度
  roughness:0.5, // 粗糙程度
  clearcoat:1,
  clearcoatRoughness:0
})
let changeColor = (color) => {
  if(nowTab ==2) {
    carBody.material.color.set(color)
  }
  if(nowTab ==1) {
    wheelsMaterial.color.set(color)
  }
  if(nowTab ==3) {
    fontCarMaterial.color.set(color)
  }
  if(nowTab ==4) {
    hoodCarMaterial.color.set(color)
  }
}
let changeTab = (tab) => {
  nowTab = tab
}
 

const count = ref(0)
let colors = ref(["red", "blue", "green", "gray", "orange", "purple"])
onMounted(() => {
  let demo = document.getElementById('demo')
  let width = demo.clientWidth
  let height = demo.clientHeight
  // 创建场景
  const scene = new THREE.Scene();
  // 创建相机
  const camera = new THREE.PerspectiveCamera(75,width/height,0.1,1000)
  // 设置相机的坐标
  camera.position.set(0,5,3)
 
  // 相机添加入场景
  scene.add(camera)


  //创建 网格平面
  let planG = new THREE.GridHelper(20,20)
  // 设置地板透明度
  planG.material.opacity = 0.5
  planG.material.transparent = true
  scene.add(planG)

 
 
  // 添加汽车模型
  /**前脸
bma.vue:57 前轮毂
bma.vue:57 右后轮毂
bma.vue:57 宝马双肾
bma.vue:57 左后轮毂
bma.vue:57 引擎盖
bma.vue:57 引擎盖_1
bma.vue:57 引擎盖_2
bma.vue:57 挡风玻璃
bma.vue:57 车身
bma.vue:57 Mesh002
bma.vue:57 Mesh002_1
bma.vue:57 进气口 */
  let gltfloader = new GLTFLoader()
  let dracoLoader = new DRACOLoader()
  dracoLoader.setDecoderPath('./draco/gltf/')
  gltfloader.setDRACOLoader(dracoLoader)
  gltfloader.load('./bwm/bmw01.glb',(gltf)=>{
    let bwa = gltf.scene
    bwa.traverse(child => {
      console.log(child.name)
      if(child.isMesh){
        if(child.name.includes('轮毂')) {
          child.material = wheelsMaterial;
          wheels.push(child)
        }
        if(child.name.includes('Mesh002')) {
          child.material = carBodyMaterial;
          carBody = child
        }
        if(child.name.includes('前脸')) {
          child.material = fontCarMaterial;
          fontCar = child
        }
        if(child.name.includes('引擎盖_1')) {
          child.material = hoodCarMaterial;
          hoodCar = child
        }
      }
    })
    //console.log(bwa)
    scene.add(bwa)
  })
  // 多增加白色光是为了车子的全部颜色呈现出来
  let newColor = (new Date()).getSeconds()/100
  const light1 = new THREE.DirectionalLight(0xffffff, 1);
  light1.position.set(0, 0, 10);
  scene.add(light1);
  const light2 = new THREE.DirectionalLight(0xffffff, 1);
  light2.position.set(0, 0, -10);
  scene.add(light2);
  const light3 = new THREE.DirectionalLight(0xffffff, 1);
  light3.position.set(10, 0, 0);
  scene.add(light3);
  const light4 = new THREE.DirectionalLight(0xffffff, 1);
  light4.position.set(-10, 0, 0);
  scene.add(light4);
  const light5 = new THREE.DirectionalLight(0xffffff, 1);
  light5.position.set(0, 10, 0);
  scene.add(light5);
  const light6 = new THREE.DirectionalLight(0xffffff, 0.3);
  light6.position.set(5, 10, 0);
  scene.add(light6);
  const light7 = new THREE.DirectionalLight(0xffffff, 0.3);
  light7.position.set(0, 10, 5);
  scene.add(light7);
  const light8 = new THREE.DirectionalLight(0xffffff, 0.3);
  light8.position.set(0, 10, -5);
  scene.add(light8);
  const light9 = new THREE.DirectionalLight(0xffffff, 0.3);
  light9.position.set(-5, 10, 0);
  scene.add(light9);
  
  //light2.color.setHSL(newColor, 1, 0.5 );

  scene.background = new THREE.Color("#ccc");
  scene.environment = new THREE.Color("#ccc");
  //初始化渲染器
  const render = new THREE.WebGLRenderer({
    antialias:true
  })
  // 设置渲染的尺寸和大小
  render.setSize(width, height)

   // 初始化控制器
  const control =  new OrbitControls(camera , render.domElement)
  //添加到dom
  demo.appendChild(render.domElement)

  

  function renderD() {
    // 使用渲染器，通过相机将场景渲染
    render.render(scene,camera)
    //浏览器api ，请求动画函数
    requestAnimationFrame(renderD)
  }
  renderD()
})

</script>



<style scoped>
.read-the-docs {
  color: #888;
}
.tools{
  position: absolute;
  width: 300px;
  top:0;
}
.tabs{
  display: flex;
}
.tab{
  cursor: pointer;
  width: 100px;
  text-align: center;
}
.colors{
  width: 100%;
  overflow: hidden;
}
.coloritem{
  cursor: pointer;
  width: 98px;
  float: left;
  margin-right: 1px;
  height: 98px;
}
</style>
