<script setup>
import { ref,onMounted } from 'vue'
import * as THREE from 'three'
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
defineProps({
  msg: String,
})

const count = ref(0)

onMounted(() => {
  let demo = document.getElementById('demo')
  let width = demo.clientWidth
  let height = demo.clientHeight
  // 创建场景
  const scene = new THREE.Scene();
  // 创建相机
  const camera = new THREE.PerspectiveCamera(75,width/height,0.1,1000)
  // 设置相机的坐标
  camera.position.set(0,0,10)
 
  // 相机添加入场景
  scene.add(camera)
  // 创建几何体
  const geometry = new THREE.BoxGeometry(2,2,2)
  // 材质
  const mesh = new THREE.MeshBasicMaterial({color: 0x00ff00})
  // 基于几何体和材质创建立方体
  const cube = new THREE.Mesh(geometry, mesh)
  scene.add(cube)
  
  //初始化渲染器
  const render = new THREE.WebGLRenderer()
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

<template>
  <div  style="width:100%;height:100%">
    <div id="demo"  style="width:100%;height:100%"></div>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
