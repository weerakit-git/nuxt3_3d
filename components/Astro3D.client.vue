<template>
  <canvas ref="experience" />
</template>

<script setup lang="ts">
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { ref, onMounted, watch } from 'vue'
import { useWindowSize } from '@vueuse/core'

const experience = ref<HTMLCanvasElement | null>(null)

// responsive
const { width, height } = useWindowSize()
const aspectRatio = computed(() => width.value / height.value)

// three vars
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
let renderer: THREE.WebGLRenderer | null = null
let mixer: THREE.AnimationMixer | null = null

const bgColor = new THREE.Color('#F54927')


// -----------------------------------------
// Mounted
// -----------------------------------------
onMounted(() => {
  init3D()
  loop()
})


// -----------------------------------------
// Init
// -----------------------------------------
function init3D() {
  scene = new THREE.Scene()
  scene.background = bgColor
  scene.fog = new THREE.Fog(bgColor, 0.1, 75)

  camera = new THREE.PerspectiveCamera(
    75,
    aspectRatio.value,
    0.1,
    1000
  )
  camera.position.set(0, 0, 4)
  scene.add(camera)

  renderer = new THREE.WebGLRenderer({
    canvas: experience.value!,
    antialias: false,          // IMPORTANT for old iPhones
    powerPreference: "low-power"
  })

  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.2)) // IMPORTANT
  renderer.setSize(width.value, height.value)

  // CHECK WEBGL CONTEXT (CRITICAL)
  const gl = renderer.getContext()
  if (!gl) {
    console.warn("WebGL context failed → disable rendering but page will NOT crash")
    renderer = null
    return
  }


  // LIGHTS (safe)
  const dir = new THREE.DirectionalLight(0xffffff, 1)
  dir.position.set(3, 3, 3)
  scene.add(dir)

  const amb = new THREE.AmbientLight(0xffffff, 0.4)
  scene.add(amb)


  // LOAD MODEL
  const loader = new GLTFLoader()

  loader.load('/model.glb', (gltf) => {
    const model = gltf.scene

    model.scale.set(1, 1, 1)
    scene!.add(model)

    if (gltf.animations.length > 0) {
      mixer = new THREE.AnimationMixer(model)
      mixer.clipAction(gltf.animations[0]).play()
    }
  })
}


// -----------------------------------------
// Resize
// -----------------------------------------
watch(aspectRatio, () => {
  if (!renderer || !camera) return
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
  renderer.setSize(width.value, height.value)
})


// -----------------------------------------
// Loop
// -----------------------------------------
function loop() {
  requestAnimationFrame(loop)

  if (!renderer || !scene || !camera) return

  if (mixer) mixer.update(0.01)

  renderer.render(scene, camera)
}
</script>
