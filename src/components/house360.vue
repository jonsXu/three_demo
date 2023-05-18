
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
  //camera.position.z = 15
  // 吧相机位置摆入物体内部，正方体是10,10,10
  camera.position.z = 0.1
  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer()
  renderer.setSize(width,height)

  const render = () => {
    renderer.render(scene,camera)
    requestAnimationFrame(render)
  }

  // /// 添加立方体实例
  const geometry = new THREE.BoxGeometry(10,10,10)
  // // 设置一个基础网格材质,可以看到一个简单的立方体
  // const material = new THREE.MeshBasicMaterial({color: 0x00ff00})
  // 使用立方体对象+材质创建物体
  // const cube = new THREE.Mesh(geometry,material)

  let arr =  ["4_l", "4_r", "4_u", "4_d", "4_b", "4_f"]
  let texturesArr = []
  arr.forEach(item => {
    // 加载纹理
    let texture = new THREE.TextureLoader().load(`./img/living/${item}.jpg`)
    // 处理顶部和底部角度错位180度
    if(item === '4_u' || item === '4_d') {
      console.log(texture)
      texture.rotation = Math.PI
      texture.center = new THREE.Vector2(0.5,0.5)
    }
    texturesArr.push(new THREE.MeshBasicMaterial({map: texture}))
  })
  const cube = new THREE.Mesh(geometry,texturesArr)
  // 对物体（cube）进行翻转
  cube.geometry.scale(-1,1,1)
  scene.add(cube)
  // 添加控制器
  const control = new OrbitControls(camera,demo)
  control.enableDamping = true
  demo.appendChild(renderer.domElement)
  render()
})

</script>



<style scoped>
.read-the-docs {
  color: #888;
}
</style>
