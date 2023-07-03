
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
  
  const scene = new THREE.Scene();
  let demo = document.getElementById('demo')
  let width = demo.clientWidth
  let height = demo.clientHeight
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  width / height,
  0.1,
  2000
);

// 设置相机位置
camera.position.set(-50, 50, 130);
// 更新摄像头宽高比例
camera.aspect = width/ height;
// 更新摄像头投影矩阵
camera.updateProjectionMatrix();

scene.add(camera);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  // 设置抗锯齿
  antialias: true,
  //   对数深度缓冲区
  logarithmicDepthBuffer: true,
});
renderer.outputEncoding = THREE.sRGBEncoding;

// 设置渲染器宽高
renderer.setSize(width, height);

// 监听屏幕的大小改变，修改渲染器的宽高，相机的比例
window.addEventListener("resize", () => {
  camera.aspect = width / height;
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
});

// 将渲染器添加到页面
document.body.appendChild(renderer.domElement);

// 实例化控制器
const controls = new OrbitControls(camera, renderer.domElement);



// 添加平面
// const planeGeometry = new THREE.PlaneGeometry(100, 100);
// const planeMaterial = new THREE.MeshBasicMaterial({
//   color: 0xffffff,
// });
// const plane = new THREE.Mesh(planeGeometry, planeMaterial);
// scene.add(plane);

// 创建一个巨大的天空球体
let texture = new THREE.TextureLoader().load("./img/islet/sky.jpg");
const skyGeometry = new THREE.SphereGeometry(1000, 60, 60);
const skyMaterial = new THREE.MeshBasicMaterial({
  map: texture,
});
skyGeometry.scale(1, 1, -1);
const sky = new THREE.Mesh(skyGeometry, skyMaterial);

scene.add(sky);

// 视频纹理
const video = document.createElement("video");
video.src = "./img/islet/sky.mp4";
video.loop = true;

window.addEventListener("click", (e) => {
  // 当鼠标移动的时候播放视频
  //   判断视频是否处于播放状态
  if (video.paused) {
    video.play();
    let texture = new THREE.VideoTexture(video);
    skyMaterial.map = texture;
    skyMaterial.map.needsUpdate = true;
  }
});

// 载入环境纹理hdr
const hdrLoader = new RGBELoader();
hdrLoader.loadAsync("./img/islet/050.hdr").then((texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = texture;
  scene.environment = texture;
});

// 添加平行光
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(-100, 100, 10);
scene.add(light);

// 创建水面
const waterGeometry = new THREE.CircleBufferGeometry(300, 64);
var water = new Water(waterGeometry, {
  textureWidth: 1024,
  textureHeight: 1024,
  color: 0xeeeeff,
  distortionScale: 8,
  waterNormals: new THREE.TextureLoader().load( './img/islet/waternormals.jpg', function ( texture ) {

    texture.wrapS = texture.wrapT = THREE.RepeatWrapping;

    } ),
  flowDirection: new THREE.Vector2(1, 1),
  scale: 1,
});
water.position.y = 3;
// 水面旋转至水平
water.rotation.x = -Math.PI / 2;
scene.add(water);

// 添加小岛模型
// 实例化gltf载入库
const loader = new GLTFLoader();
// 实例化draco载入库
const dracoLoader = new DRACOLoader();
// 添加draco载入库
dracoLoader.setDecoderPath("./draco/");
// 添加draco载入库
loader.setDRACOLoader(dracoLoader);

loader.load("./img/islet/model/island2.glb", (gltf) => {
  scene.add(gltf.scene);
});
function render() {
  water.material.uniforms[ 'time' ].value += 1.0 / 60.0;
  // 渲染场景
  renderer.render(scene, camera);
  // 引擎自动更新渲染器
  requestAnimationFrame(render);
}
render();
})

</script>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
