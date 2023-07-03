
<template>
  <div  style="width:100%;height:100%">
    <div id="demo"  style="width:100%;height:100%"></div>
  </div>
</template>
<script setup>
/**
 * 360度全方位vr看房，其实就是
 */
import { ref,onMounted } from 'vue'
import * as THREE from 'three'
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";  // 加载hdr就需要这个纹理加载器
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';




onMounted(() => {
  const count = ref(0)
  // 初始化场景
  const scene = new THREE.Scene();
  // 初始化相机,(3D场景，一般用透视相机)
  let demo = document.getElementById('demo')
    let width = demo.clientWidth
    let height = demo.clientHeight
  const camera = new THREE.PerspectiveCamera(75,width/height,0.1,1000)
  // 设置相机的位置，是为了让我们在物体内部查看
  camera.position.z = 15
  // 吧相机位置摆入物体内部，正方体是10,10,10
  //camera.position.z = 0.1
  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer()
  renderer.setSize(width,height)

  const render = () => {
    renderer.render(scene,camera)
    requestAnimationFrame(render)
  }

  // /// 添加立方体实例
  const geometry = new THREE.SphereGeometry(5,32,16)
 
  let loader = new RGBELoader ()
  loader.loadAsync('./img/hdr/Living.hdr').then(texture =>{
     // 加载hdr贴图，需要使用经纬贴图,不能当作一个普通图片贴图
    texture.mapping = THREE.EquirectangularReflectionMapping  // 设置经纬度贴图
    const material = new THREE.MeshBasicMaterial({map: texture}) 
    const cube = new THREE.Mesh(geometry,material)
   // cube.geometry.scale(1, 1, -1);
    
    scene.add(cube)
    scene.background = texture
    // 添加控制器
    const control = new OrbitControls(camera,demo)
    control.enableDamping = true
    demo.appendChild(renderer.domElement)
    render()
  })
 
  
})

</script>



<style scoped>
.read-the-docs {
  color: #888;
}
</style>
