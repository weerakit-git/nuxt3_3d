<template>
  <div style="position: relative; width: 100%; height: 100vh;">
    
    <canvas 
      ref="experience" 
      style="position: absolute; inset: 0; z-index: 1;"
    />

    <div 
      style="
      display: flex;
      flex-direction: column;
      gap: 10px;
        position: absolute;
        top: 20px;
        left: 20px;
        z-index: 10;
        color: salmon;
        font-size: 24px;
      "
    >
    <span> HTML รันได้เถอะสาธุ อยากนอนแล้ว</span>
    <img src="/public/image1.jpg" alt="" >
     

    </div>

  </div>
</template>


<script setup lang="ts">
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { ref, onMounted, watch } from 'vue'
import { useWindowSize } from '@vueuse/core'

const experience = ref<HTMLCanvasElement | null>(null)
const { width, height } = useWindowSize()
const aspectRatio = computed(() => width.value / height.value)

// three vars
let scene, camera, renderer, mixer

// ------------------------------------------
// safe WebGL check (แทน WebGL.js ที่หายไป)
// ------------------------------------------
function isWebGLAvailable() {
  try {
    const canvas = document.createElement('canvas')
    return !!(
      window.WebGLRenderingContext &&
      (canvas.getContext('webgl') || canvas.getContext('experimental-webgl'))
    )
  } catch (e) {
    return false
  }
}

onMounted(() => {
  if (!isWebGLAvailable()) {
    console.warn("WebGL not supported on this device")
    return
  }

  try {
    init3D()
    loop()
  } catch (err) {
    console.warn("Renderer init failed:", err)
  }
})

function init3D() {
  scene = new THREE.Scene()

  camera = new THREE.PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
  camera.position.set(0, 0, 4)
  scene.add(camera)

  renderer = new THREE.WebGLRenderer({
    canvas: experience.value!,
    antialias: true,
  })
  renderer.setSize(width.value, height.value)

  const loader = new GLTFLoader()
  loader.load('/model.glb', (gltf) => {
    const model = gltf.scene
    scene.add(model)

    if (gltf.animations.length > 0) {
      mixer = new THREE.AnimationMixer(model)
      mixer.clipAction(gltf.animations[0]).play()
    }
  })
}

watch(aspectRatio, () => {
  if (!camera || !renderer) return
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
  renderer.setSize(width.value, height.value)
})

function loop() {
  requestAnimationFrame(loop)
  if (mixer) mixer.update(0.01)
  renderer?.render(scene, camera)
}
</script>

<style scoped>
img{
  width: 250px;
  height: 250px;
}
</style>
