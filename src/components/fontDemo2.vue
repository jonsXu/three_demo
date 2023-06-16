
<template>
  <div  style="width:100%;height:100%">
    <div id="demo"  style="width:100%;height:100%"></div>
  </div>
</template>
<script setup>
import { ref,onMounted } from 'vue'
import * as THREE from 'three'
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
// 轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';
defineProps({
  msg: String,
})

const count = ref(0)

onMounted(() => {
  let demo = document.getElementById('demo')
  let widthD = demo.clientWidth
  let heightD = demo.clientHeight
  
  THREE.Cache.enabled = true;

			let container;

			let camera, cameraTarget, scene, renderer;

			let group, textMesh1, textMesh2, textGeo, materials;


			let text = 'zuoyebang',

				bevelEnabled = true,

				font = undefined;

			const height = 20,
				size = 70,
				hover = 30,

				curveSegments = 4,
				bevelThickness = 2,
				bevelSize = 1.5;

			const mirror = true;


			let targetRotation = 0;

			init();
			animate();

			function init() {

				container = demo;
				//document.body.appendChild( container );

				// CAMERA

				camera = new THREE.PerspectiveCamera( 30, widthD / heightD, 1, 1500 );
				camera.position.set( 0, 400, 700 );

				//cameraTarget = new THREE.Vector3( 0, 150, 0 );

				// SCENE

				scene = new THREE.Scene();
				scene.background = new THREE.Color( 0x000000 );
				scene.fog = new THREE.Fog( 0x000000, 250, 1400 );

				// LIGHTS

				const dirLight = new THREE.DirectionalLight( 0xffffff, 0.125 );
				dirLight.position.set( 0, 0, 1 ).normalize();
				scene.add( dirLight );

				const pointLight = new THREE.PointLight( 0xffffff, 1.5 );
				pointLight.color.setHSL( Math.random(), 1, 0.5 );
				pointLight.position.set( 0, 100, 90 );
				scene.add( pointLight );

				materials = [
					new THREE.MeshPhongMaterial( { color: 0xffffff, flatShading: true } ), // front
					new THREE.MeshPhongMaterial( { color: 0xffffff } ) // side
				];

				group = new THREE.Group();
				group.position.y = 100;

				//scene.add( group );

				

				const plane = new THREE.Mesh(
					new THREE.PlaneGeometry( 10000, 10000 ),
					new THREE.MeshBasicMaterial( { color: 0xffffff, opacity: 0.5, transparent: true } )
				);
				plane.position.y = 100;
				plane.rotation.x = - Math.PI / 2;
				//scene.add( plane );

				// RENDERER

				renderer = new THREE.WebGLRenderer( { antialias: true } );
				//renderer.setPixelRatio( window.devicePixelRatio );
				renderer.setSize( widthD, heightD );
				container.appendChild( renderer.domElement );
        const control =  new OrbitControls(camera , renderer.domElement)
				// EVENTS

				//container.style.touchAction = 'none';
        const loader = new FontLoader();
				loader.load( './font/helvetiker_bold.typeface.json', function ( response ) {

					font = response;

					createText();

				} );
        //

			}


			function createText() {

				textGeo = new TextGeometry( text, {

					font: font,

					size: size,
					height: height,
					curveSegments: curveSegments,

					bevelThickness: bevelThickness,
					bevelSize: bevelSize,
					bevelEnabled: bevelEnabled

				} );

				textGeo.computeBoundingBox();

				//const centerOffset = - 0.5 * ( textGeo.boundingBox.max.x - textGeo.boundingBox.min.x );

				textMesh1 = new THREE.Mesh( textGeo, materials );

				//textMesh1.position.x = centerOffset;
				//textMesh1.position.y = hover;
				//textMesh1.position.z = 0;

				//textMesh1.rotation.x = 0;
				//textMesh1.rotation.y = Math.PI * 2;

				scene.add( textMesh1 );

				// if ( mirror ) {

				// 	textMesh2 = new THREE.Mesh( textGeo, materials );

				// 	textMesh2.position.x = centerOffset;
				// 	textMesh2.position.y = - hover;
				// 	textMesh2.position.z = height;

				// 	textMesh2.rotation.x = Math.PI;
				// 	textMesh2.rotation.y = Math.PI * 2;

				// 	group.add( textMesh2 );

				// }

			}



			function animate() {
        render();
				requestAnimationFrame( animate );

				

			}

			function render() {

				//group.rotation.y += ( targetRotation - group.rotation.y ) * 0.05;

				//camera.lookAt( cameraTarget );

				renderer.clear();
				renderer.render( scene, camera );

			}
})

</script>



<style scoped>
.read-the-docs {
  color: #888;
}
</style>
