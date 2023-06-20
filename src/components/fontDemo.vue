
<template>
  <input v-model="state.text" @input="changeText" style="width: 0;
    height: 0;
    border: 0;
    padding: 0;
    position: absolute;" id="input1"/>
  <div  style="width:100%;height:100%">
    <div id="demo"  style="width:100%;height:100%"></div>
  </div>
</template>
<script setup>
import { ref,onMounted, unref,reactive } from 'vue'
import * as THREE from 'three'
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
defineProps({
  msg: String,
})
var createText = null
var refresh = null
function changeText (){
  refresh()
}
const state = reactive({
    text: '作业帮',
  })
  var meshShadow  = null
  var mesh = null
function keyDown(even) {
  // 回车
  if (even.keyCode === 13) {
    document.getElementById('input1').focus()
  }
}
window.addEventListener('keydown', keyDown)
onMounted(() => {
  
  let demo = document.getElementById('demo')
  let width = demo.clientWidth
  let height = demo.clientHeight

  let text = state.text
  let pointXDown = 0
  let pointX = 0
  let lineRotation = 0
  let targetRotationOnPointerDown = 0
  function onPointDown(event) {
    pointXDown = event.clientX - width/2 // 鼠标点击的位置离中心点的距离
    targetRotationOnPointerDown = lineRotation
    demo.addEventListener('pointermove',onPointMove)
    demo.addEventListener('pointerup',onMouseUp)
    
  }
  function onPointMove(event) {
    pointX = event.clientX - width/2 // 鼠标移动的位置距离中心点的距离
    lineRotation = targetRotationOnPointerDown + (pointX - pointXDown) * 0.01 // 点光源旋转点距离 = 当前电光源点位置+  移动距离-原来点距离
  }
  function onMouseUp() {
    debugger
    demo.removeEventListener( 'pointermove', onPointMove );
    demo.removeEventListener( 'pointerup', onMouseUp );
  }
  demo.addEventListener('mousedown',onPointDown)
  
  // 创建场景
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x000000)
  scene.fog = new THREE.Fog( 0x000000, 250, 1400 ); // 
  // 创建相机
  const camera = new THREE.PerspectiveCamera(30,width/height,1,1500)
  // 设置相机的坐标
  camera.position.set(0,100,500)
  //let cameraTarget = new THREE.Vector3( 0, 150, 0 );
  // 相机添加入场景
  scene.add(camera)

  //初始化渲染器
  const render = new THREE.WebGLRenderer()
  // 设置渲染的尺寸和大小
  render.setSize(width, height)
  render.setPixelRatio( window.devicePixelRatio );
   // 初始化控制器
  //const control =  new OrbitControls(camera , render.domElement)
  //添加到dom
  demo.appendChild(render.domElement)
  
  // 创建光源
  const light = new THREE.DirectionalLight( 0xffffff,0.125);
  light.position.set( 0,0,1 ).normalize()
  scene.add( light );
  // 点光源
  const pointLight = new THREE.PointLight( 0xffffff, 1.5 );
  pointLight.color.setHSL( Math.random(), 1, 0.5 );
  pointLight.position.set( 0, 100, 120 );
  scene.add( pointLight );
  // 创建平面
  let plan = new THREE.PlaneGeometry(10000,10000)
  let materialPlan = new THREE.MeshBasicMaterial({color:0xffffff,opacity: 0.5, transparent: true})
  let plane = new THREE.Mesh(plan,materialPlan)
  plane.rotation.x = - Math.PI * 0.5
  plane.position.y = 40;
  //plane.position.set(100,100,100)
  scene.add(plane)

  let group  = new THREE.Group()
  group.position.y = 100;
  scene.add(group)
  // 创建字体

  // 创建字体几何体
  
  let font = undefined
  let fontloader = new FontLoader()

  fontloader.load('./font/china.json',function(res) {
    font = res
    createText()
  })
  
  createText = function  () {
    let geometry = new TextGeometry(state.text, {
      font: font,
      size: 50,
      height: 20,
      curveSegments: 4,
      bevelEnabled: true,
      bevelThickness: 2,
      bevelSize: 1.5,
      bevelSegments: 2
    })
    geometry.computeBoundingBox()
    let material = new THREE.MeshPhongMaterial({
      color: 0xffffff,
      flatShading: true
    })
    let materials = [
					new THREE.MeshPhongMaterial( { color: 0xffffff, flatShading: true } ), // front
					new THREE.MeshPhongMaterial( { color: 0xffffff ,flatShading:true} ) // side
				];
    mesh = new THREE.Mesh(geometry, materials)
    mesh.position.x = -120;
    mesh.position.y = -30;
    mesh.position.z = 0;

    //mesh.rotation.x = 0;
    //mesh.rotation.y = Math.PI * 2;
    group.add( mesh );
    //scene.add(mesh)
    // 做字体倒影
    meshShadow = new THREE.Mesh(geometry, materials)
    meshShadow.position.x = -120;
    meshShadow.position.y = -65;
    meshShadow.position.z = 10;
    meshShadow.rotation.x = Math.PI;
    group.add( meshShadow );
    //scene.add(meshShadow)
  }
  refresh = function () {
    group.remove(mesh)
    group.remove(meshShadow)
    createText()
  }
  function renderD() {
    //颜色渐变
    pointLight.color.setHSL((new Date()).getSeconds()/100, 1, 0.5 );
    //camera.lookAt( cameraTarget );
    
    group.rotation.y += ( lineRotation - group.rotation.y ) * 0.005;
    render.clear()

    // 使用渲染器，通过相机将场景渲染
    render.render(scene,camera)
    //浏览器api ，请求动画函数
    
  }
  function animation () {
    requestAnimationFrame(animation)
    renderD()
  }
  animation()
})


</script>



<style scoped>
.read-the-docs {
  color: #888;
}
</style>
