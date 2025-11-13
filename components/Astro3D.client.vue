<template>
  <div id="container3D"></div>
</template>

<script setup>
import { onMounted } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'

let scene, camera, renderer
let model, mixer

onMounted(() => {
  // Scene
  scene = new THREE.Scene()

  // Camera
  camera = new THREE.PerspectiveCamera(
    30,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.z = 10
  camera.lookAt(0, 0, 0)

  // Renderer
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.toneMapping = THREE.LinearToneMapping
  renderer.toneMappingExposure = Math.pow(2, -10)
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)

  document.getElementById("container3D").appendChild(renderer.domElement)

  // Lights
  const dirLight = new THREE.DirectionalLight(0xffffff, 1)
  dirLight.position.set(1, 1, 1)
  scene.add(dirLight)

  const ambient = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambient)

  // Loader
  const loader = new GLTFLoader()

  loader.load("/model.glb", (gltf) => {
    model = gltf.scene

    const box = new THREE.Box3().setFromObject(model)
    const center = new THREE.Vector3()
    box.getCenter(center)
    model.position.sub(center)

    model.scale.set(1, 1, 1)
    scene.add(model)

    mixer = new THREE.AnimationMixer(model)
    if (gltf.animations.length > 0) {
      mixer.clipAction(gltf.animations[0]).play()
    }
  })

  // Animate
  function animate() {
    requestAnimationFrame(animate)
    if (mixer) mixer.update(0.01)
    renderer.render(scene, camera)
  }
  animate()

  // Resize
  window.addEventListener("resize", () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })
})
</script>

<style>
#container3D {
  position: fixed;
  inset: 0;
}
</style>
