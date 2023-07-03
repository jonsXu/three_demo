
<template>
  <div  style="width:100%;height:100%; position:relative;overflow: hidden;">
    <div class="show" :style="`transform: translate3d(0, ${-nowIndex * 100}%, 0)`">
      <div  v-for="(item,index) in scenes" :key="item.text" style="width:100%; height:100%;" 
      >
      {{ item.text }}</div>
    </div>
    <div id="demo"  style="width:100%;height:100%"></div>
    
  </div>
</template>
<script setup>
import { ref,onMounted } from 'vue'
import * as THREE from 'three'
import { Water } from "three/examples/jsm/objects/Water";
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader';
//导入gltf的载入库
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
// 因为我们的模型是压缩过的，所以需要导入解压的库
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';
import { gsap } from 'gsap';
defineProps({
  msg: String,
})


let camera = null
let control = null
// 创建场景
const scene = new THREE.Scene();

onMounted(() => {
  let demo = document.getElementById('demo')
  let width = demo.clientWidth
  let height = demo.clientHeight
  
  // 创建相机
  camera = new THREE.PerspectiveCamera(75,width/height,0.1,1000)
  // 设置相机的坐标
  camera.position.set(-10,6,63)
  //camera.aspect = width/ height
  camera.updateProjectionMatrix()
 
  // 相机添加入场景
  scene.add(camera)

  // 载入环境纹理hdr
  const hdrLoader = new RGBELoader();
  hdrLoader.loadAsync("./happynewyear/sky.hdr").then((texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.background = texture
    //scene.background = new THREE.Color("#ccc");
    scene.environment = texture;
    //scene.environment = new THREE.Color("#ccc");
  });

  //创建一个水面
  const circleBufferGeometry = new THREE.CircleBufferGeometry(1000,100)
  const water = new Water(circleBufferGeometry,
    {
      textureWidth: 1024, //纹理宽度
      textureHeight: 1024, //纹理高度
      color: 0xeeeeff, // 水的颜色
      waterNormals: new THREE.TextureLoader().load( './img/islet/waternormals.jpg', function ( texture ) {
        texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
      } ),
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

 // 加载冬日小岛模型

  const loader = new GLTFLoader()
  // 实例化解压库
  const dracoLoader = new DRACOLoader()
  // 添加解压库
  dracoLoader.setDecoderPath('/draco/')
  loader.setDRACOLoader(dracoLoader)
  // 加载小岛
  loader.load('./happynewyear/smallhouse.glb',(gltf)=>{
    console.log(gltf)
    gltf.scene.position.y=6
    gltf.scene.rotation.y = Math.PI
    scene.add(gltf.scene)
    gltf.scene.traverse(children => {
      if(children.isMesh){
        // 物体能够投射阴影
        children.castShadow = true
        // 物体能够接收阴影
        children.receiveShadow = true
      }
    })
  })
  // 加载高达
  // loader.load('./happynewyear/pose.glb',(gltf)=>{
  //   console.log(gltf)
  //   gltf.scene.position.set(-2,3,48)
  //   gltf.scene.rotation.y = Math.PI 
  //   gltf.scene.traverse(children => {
  //     if(children.name === 'Plane000'){
  //       children.visible = false
  //     }
  //     if(children.isMesh){
  //       // 物体能够投射阴影
  //       children.castShadow = true
  //       // 物体能够接收阴影
  //       children.receiveShadow = true
  //     }
  //   })
  //   scene.add(gltf.scene)
  // })
 
  // 制作室内点光源

  const light = new THREE.PointLight(0xffffff, 1);
  light.position.set(2, 3 ,48);
  light.castShadow = true;
  scene.add(light);

  //初始化渲染器
  const render = new THREE.WebGLRenderer({
    antialias:true,
    shadowMap: {
      enabled: true,
      //autoUpdate: true,
    },
  })
  // 设置渲染的尺寸和大小
  render.setSize(width, height)
  // 设置色调映射
  render.toneMapping = THREE.ACESFilmicToneMapping
  render.outputEncoding = THREE.sRGBEncoding

   // 初始化控制器
  control =  new OrbitControls(camera , render.domElement)
  //添加到dom
  demo.appendChild(render.domElement)

  

  function renderD() {
    // water.material.uniforms[ 'time' ].value += 1.0 / 60.0;
    // 使用渲染器，通过相机将场景渲染
    render.render(scene,camera)
    //浏览器api ，请求动画函数
    requestAnimationFrame(renderD)
  }
  renderD()
})
// 使用补间动画进行移动，创建两个补间动画的实例
let timeline1 = gsap.timeline()
let timeline2 = gsap.timeline()
function changeTarget(position,target){
  timeline1.to(camera.position,{
    x:position.x,
    y:position.y,
    z:position.z,
    duration:1,
    ease:'power2.inOut'
  })
  timeline2.to(control.target,{
    x:target.x,
    y:target.y,
    z:target.z,
    duration:1,
    ease:'power2.inOut'
  })
}
let roofPosition = {
  x: 1.4261975288391113, y: 7.708840250968933, z: -48.628902435302734
}
let treePosition = {
  x: 5, y: 7.202917098999023, z: 0
}
// 定义不同的场景
let scenes = [
  {
    text:'Happy New Year',
    callback:function(){
      // 回调函数执行的是将相机的位置进行移动，将控制器的聚焦位置切换
      console.log('Happy New Year')
      changeTarget(new THREE.Vector3(-10,6,63),new THREE.Vector3(-2,3,48))
    }
  },
  {
    text:'我在房顶',
    callback:function(){
      console.log('场景2')
      changeTarget(new THREE.Vector3(4,6,66),new THREE.Vector3(-8.4261975288391113,  7.708840250968933,  53.628902435302734))
    }
  },
  {
    text:'满天爱心',
    callback:function(){
      console.log('满天爱心')
      makeHeart()
      changeTarget(new THREE.Vector3(-15,6,80),new THREE.Vector3(-4,3,48))
    }
  },
  {
    text:'这有一棵树',
    callback:function(){
      console.log('场景4')
      changeTarget(new THREE.Vector3(5,20,0),new THREE.Vector3(5,7,0))
    }
  },
]
let flag = false
let nowIndex = ref(0) // 当前索引
// 监听鼠标滚轮事件
window.addEventListener('wheel',function(e){
  if(flag) return
  console.log(e)
  // 每次向上滚动，就给当前索引+1，如果nowIndex超过了scenes.length-1,则重置为0
  if(e.deltaY>0){
    flag = true
    nowIndex.value++
    if(nowIndex.value>scenes.length-1){
      nowIndex.value=0
      backHeart()
    }
    scenes[nowIndex.value].callback()
  }
  setTimeout(() => {
    flag = false
  }, 1000);
},false)

//创建满天的星星
let startPositions = [] // 记录星星开始位置的坐标
let endPositions = [] // 记录星星结束为止的坐标
// 两个坐标用于最后复原
let stars = new THREE.InstancedMesh(
  new THREE.SphereGeometry(0.3, 32, 32),
  new THREE.MeshStandardMaterial({
    color: 0xffffff,
    transparent: true,
    emissive: 0xffffff,
    emissiveIntensity: 10
  }),
  70
)
for (let i = 0; i < 70; i++) {
  let x = Math.random()*100-50
  let y = Math.random()*100-50
  let z = Math.random()*100-50
  startPositions.push(new THREE.Vector3(x,y,z))
  let matriax = new THREE.Matrix4() // 创建一个矩阵，用于生成星星的坐标
  matriax.setPosition(x,y,z)
  stars.setMatrixAt(i,matriax)
}
scene.add(stars)
 
// 创建一个路径实例，用于生成爱心路径
let shape = new THREE.Shape()
shape.moveTo(25, 25)
shape.bezierCurveTo(25, 25, 25, 0, 0, 0)
shape.bezierCurveTo(-30, 0, -30, 35, -30, 35)
shape.bezierCurveTo(-30, 55, -10, 77, 25, 95)
shape.bezierCurveTo(60, 77, 80, 55, 80, 35)
shape.bezierCurveTo(80, 35, 80, 0, 50, 0)
shape.bezierCurveTo(35, 0, 25, 25, 25, 25)

// 创建一个中心坐标，让爱心出现在目标位置
let center = new THREE.Vector3(-4, 5, 60)
for (let i = 0; i < 70; i++) {
  // 把路线平均分成70份，每一份都是1一个点
  let point = shape.getPointAt(i / 70)
  //获取点点坐标，就是最终的位置。每个点点坐标加上center点对应坐标，就是爱心平移后的坐标
  endPositions.push(
    new THREE.Vector3(
      point.x*0.1+center.x,
      point.y*0.1+center.y,
      center.z
    )
  )
}
function makeHeart(){
  let params = {
    time: 0
  }
  gsap.to(params,{
    time:1,
    duration:1,
    onUpdate:function(){
      for(let i = 0;i<70;i++){
        let x = startPositions[i].x+(endPositions[i].x-startPositions[i].x)*params.time
        let y = startPositions[i].y+(endPositions[i].y-startPositions[i].y)*params.time
        let z = startPositions[i].z+(endPositions[i].z-startPositions[i].z)*params.time
        // 创建一个矩阵
        let matriax = new THREE.Matrix4()
        matriax.setPosition(x,y,z)
        stars.setMatrixAt(i,matriax)
      }
      stars.instanceMatrix.needsUpdate = true
    }
  })
}
function backHeart(){
  let params = {
    time: 0
  }
  gsap.to(params,{
    time:1,
    duration:1,
    onUpdate:function(){
      for(let i = 0;i<70;i++){
        let x = endPositions[i].x-(endPositions[i].x-startPositions[i].x)*params.time
        let y = endPositions[i].y-(endPositions[i].y-startPositions[i].y)*params.time
        let z = endPositions[i].z-(endPositions[i].z-startPositions[i].z)*params.time
        // 创建一个矩阵
        let matriax = new THREE.Matrix4()
        matriax.setPosition(x,y,z)
        stars.setMatrixAt(i,matriax)
      }
      stars.instanceMatrix.needsUpdate = true
    }
  })
}
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
.show{
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  transition: all 1s;
  font-size: 53px;
  color: white;
  text-align: left;

}
</style>
