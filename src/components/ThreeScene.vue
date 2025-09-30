<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
import { ref, onMounted, onUnmounted } from 'vue'

const doorWidth = ref(1.2)
const doorHeight = ref(2.5)

let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let controls: OrbitControls
let door: THREE.Group
let shape1: THREE.Mesh
let shape2: THREE.Mesh
let animationFrameId: number

const init = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x87CEEB)
  scene.fog = new THREE.Fog(0x87CEEB, 10, 20)

  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.set(0, 1.5, 5)

  const container = document.getElementById('scene-container')!
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.clientWidth, container.clientHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.setPixelRatio(window.devicePixelRatio)
  container.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  addLights()
  addEnvironment()
  addGeometricShapes()
  createDoor()

  window.addEventListener('resize', onWindowResize)
  animate()
}

const addLights = () => {
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
  directionalLight.position.set(5, 10, 7)
  directionalLight.castShadow = true
  directionalLight.shadow.mapSize.width = 1024
  directionalLight.shadow.mapSize.height = 1024
  scene.add(directionalLight)

  const pointLight = new THREE.PointLight(0xff7f00, 1, 100)
  pointLight.position.set(0, 3, 0)
  pointLight.castShadow = true
  scene.add(pointLight)
}

const addEnvironment = () => {
  const groundGeometry = new THREE.PlaneGeometry(20, 20)
  const groundMaterial = new THREE.MeshStandardMaterial({ 
    color: 0x228B22,
    roughness: 0.8,
    metalness: 0.2
  })
  const ground = new THREE.Mesh(groundGeometry, groundMaterial)
  ground.rotation.x = -Math.PI / 2
  ground.position.y = -0.05
  ground.receiveShadow = true
  scene.add(ground)

  const skyGeometry = new THREE.SphereGeometry(15, 32, 32)
  const skyMaterial = new THREE.MeshBasicMaterial({
    color: 0x87CEEB,
    side: THREE.BackSide
  })
  const sky = new THREE.Mesh(skyGeometry, skyMaterial)
  scene.add(sky)
}

const addGeometricShapes = () => {
  const torusKnotGeometry = new THREE.TorusKnotGeometry(0.8, 0.3, 128, 32)
  const torusKnotMaterial = new THREE.MeshStandardMaterial({
    color: 0xff0000,
    roughness: 0.1,
    metalness: 0.9
  })
  shape1 = new THREE.Mesh(torusKnotGeometry, torusKnotMaterial)
  shape1.position.set(-3, 1.5, 0)
  shape1.castShadow = true
  shape1.receiveShadow = true
  scene.add(shape1)

  const icosahedronGeometry = new THREE.IcosahedronGeometry(1, 0)
  const icosahedronMaterial = new THREE.MeshStandardMaterial({
    color: 0x0000ff,
    roughness: 0.3,
    metalness: 0.7
  })
  shape2 = new THREE.Mesh(icosahedronGeometry, icosahedronMaterial)
  shape2.position.set(3, 1.5, 0)
  shape2.castShadow = true
  shape2.receiveShadow = true
  scene.add(shape2)
}

const createDoor = () => {
  if (door) {
    scene.remove(door)
  }

  door = new THREE.Group()

  const woodMaterial = new THREE.MeshStandardMaterial({
    color: 0x8B4513,
    roughness: 0.7,
    metalness: 0.1
  })

  const frameWidth = doorWidth.value + 0.2
  const frameHeight = doorHeight.value + 0.2
  
  const topFrameGeometry = new THREE.BoxGeometry(frameWidth, 0.1, 0.1)
  const topFrame = new THREE.Mesh(topFrameGeometry, woodMaterial)
  topFrame.position.set(0, doorHeight.value / 2, 0)
  topFrame.castShadow = true
  door.add(topFrame)

  const bottomFrameGeometry = new THREE.BoxGeometry(frameWidth, 0.1, 0.1)
  const bottomFrame = new THREE.Mesh(bottomFrameGeometry, woodMaterial)
  bottomFrame.position.set(0, -doorHeight.value / 2, 0)
  bottomFrame.castShadow = true
  door.add(bottomFrame)

  const leftFrameGeometry = new THREE.BoxGeometry(0.1, doorHeight.value, 0.1)
  const leftFrame = new THREE.Mesh(leftFrameGeometry, woodMaterial)
  leftFrame.position.set(-doorWidth.value / 2 - 0.05, 0, 0)
  leftFrame.castShadow = true
  door.add(leftFrame)

  const rightFrameGeometry = new THREE.BoxGeometry(0.1, doorHeight.value, 0.1)
  const rightFrame = new THREE.Mesh(rightFrameGeometry, woodMaterial)
  rightFrame.position.set(doorWidth.value / 2 + 0.05, 0, 0)
  rightFrame.castShadow = true
  door.add(rightFrame)

  const panelGeometry = new THREE.BoxGeometry(doorWidth.value, doorHeight.value, 0.05)
  const panelMaterial = new THREE.MeshStandardMaterial({
    color: 0xA0522D,
    roughness: 0.6,
    metalness: 0.0
  })
  const panel = new THREE.Mesh(panelGeometry, panelMaterial)
  panel.castShadow = true
  panel.receiveShadow = true
  door.add(panel)

  const handleGeometry = new THREE.CylinderGeometry(0.05, 0.05, 0.01, 16)
  const handleMaterial = new THREE.MeshStandardMaterial({
    color: 0xFFD700,
    roughness: 0.2,
    metalness: 0.9
  })
  const handle = new THREE.Mesh(handleGeometry, handleMaterial)
  handle.rotation.x = Math.PI / 2
  handle.position.set(doorWidth.value / 3, 0, 0.03)
  handle.castShadow = true
  door.add(handle)

  door.position.set(0, doorHeight.value / 2, -2)
  door.castShadow = true
  scene.add(door)
}

const updateDoor = () => {
  createDoor()
}

const onWindowResize = () => {
  const container = document.getElementById('scene-container')!
  camera.aspect = container.clientWidth / container.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(container.clientWidth, container.clientHeight)
}

const animate = () => {
  animationFrameId = requestAnimationFrame(animate)
  
  if (shape1) {
    shape1.rotation.x += 0.01
    shape1.rotation.y += 0.01
  }
  
  if (shape2) {
    shape2.rotation.x += 0.01
    shape2.rotation.y -= 0.01
  }
  
  controls.update()
  renderer.render(scene, camera)
}

onUnmounted(() => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
  }
  window.removeEventListener('resize', onWindowResize)
  if (renderer) {
    renderer.dispose()
  }
})

onMounted(() => {
  init()
})
</script>

<template>
  <div class="three-scene-container">
    <div id="scene-container"></div>
    
    <div class="controls">
      <h3>Управление дверью</h3>
      <div class="control-group">
        <label for="width">Ширина: {{ doorWidth }}м</label>
        <input 
          id="width" 
          type="range" 
          min="1" 
          max="4" 
          step="0.1" 
          v-model.number="doorWidth" 
          @input="updateDoor"
        />
      </div>
      
      <div class="control-group">
        <label for="height">Высота: {{ doorHeight }}м</label>
        <input 
          id="height" 
          type="range" 
          min="2" 
          max="4" 
          step="0.1" 
          v-model.number="doorHeight" 
          @input="updateDoor"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.three-scene-container {
  position: relative;
  width: 100%;
  height: 100vh;
}

#scene-container {
  width: 100%;
  height: 100%;
}

.controls {
  position: absolute;
  top: 20px;
  left: 20px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 15px;
  border-radius: 8px;
  z-index: 100;
  min-width: 250px;
}

.control-group {
  margin-bottom: 15px;
}

.control-group label {
  display: block;
  margin-bottom: 5px;
}

.control-group input {
  width: 100%;
}
</style>