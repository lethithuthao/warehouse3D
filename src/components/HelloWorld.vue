<template>
  <div class="container">
    <div class="scene-container" ref="sceneContainer"></div>
    <div class="table-container">
      <table class="inventory-table">
        <thead>
          <tr>
            <th>LOCATION</th>
            <th>ITEM NO</th>
            <th>DESCRIPTION</th>
            <th>COLOR</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, idx) in warehouseData" :key="idx">
            <td>{{ item.LOCATION }}</td>
            <td>{{ item['\\sITEM NO'] }}</td>
            <td>{{ item['ITEM DESCRIPTION'] }}</td>
            <td>{{ item['COLOR_CASES DAMAGED'] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

// Dữ liệu inventory và layout
import warehouseData from '@/assets/inventory_converted.json'
import layoutData from '@/assets/layout.json'

const sceneContainer = ref(null)

onMounted(() => {
  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0xf0f0f0)

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 2000)
  camera.position.set(100, 150, 200)

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth * 0.7, window.innerHeight) // 70% width for Three.js scene
  sceneContainer.value.appendChild(renderer.domElement)

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(10, 20, 10)
  scene.add(directionalLight)

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambientLight)

  const grid = new THREE.GridHelper(1000, 500)
  scene.add(grid)

  const axesHelper = new THREE.AxesHelper(50)
  scene.add(axesHelper)

  // Scale hệ toạ độ nếu layout quá lớn
  const scaleFactor = 0.1

  // Map layout theo LOCATION
  const layoutMap = {}
  layoutData.forEach(loc => {
    layoutMap[loc.LOCATION] = loc
  })

  // Tính tâm kho
  let minX = Infinity, maxX = -Infinity
  let minY = Infinity, maxY = -Infinity
  let minZ = Infinity, maxZ = -Infinity

  layoutData.forEach(loc => {
    const x = parseFloat(loc.X)
    const y = parseFloat(loc.Y)
    const z = parseFloat(loc.Z)
    if (x < minX) minX = x
    if (x > maxX) maxX = x
    if (y < minY) minY = y
    if (y > maxY) maxY = y
    if (z < minZ) minZ = z
    if (z > maxZ) maxZ = z
  })

  const centerX = ((minX + maxX) / 2) * scaleFactor
  const centerY = ((minZ + maxZ) / 2) * scaleFactor // chiều cao là trục Y trong Three.js
  const centerZ = ((minY + maxY) / 2) * scaleFactor

  const boxGeometry = new THREE.BoxGeometry(1, 1, 1)

  const groupedItems = {}
  warehouseData.forEach(item => {
    const loc = item.LOCATION
    if (!groupedItems[loc]) groupedItems[loc] = []
    groupedItems[loc].push(item)
  })

  Object.entries(groupedItems).forEach(([loc, items]) => {
    const layout = layoutMap[loc]
    if (!layout) return

    const baseX = parseFloat(layout.X) * scaleFactor || 0
    const baseZ = parseFloat(layout.Y) * scaleFactor || 0
    const baseY = parseFloat(layout.Z) * scaleFactor || 0
    const width = parseFloat(layout.WIDTH) * scaleFactor || 1
    const height = parseFloat(layout.HEIGHT) * scaleFactor || 1
    const depth = parseFloat(layout.DEPTH) * scaleFactor || 1

    items.forEach((item, index) => {
      const baseColor = item["COLOR_CASES DAMAGED"]
      let color
      if (baseColor === "red") {
        color = index === 0 ? 0xff0000 : 0xff6666
      } else if (baseColor === "blue") {
        color = index === 0 ? 0x0000ff : 0x6666ff
      } else {
        color = index === 0 ? 0x00ff00 : 0x99ff99
      }

      const material = new THREE.MeshStandardMaterial({ color })
      const cube = new THREE.Mesh(boxGeometry, material)
      cube.scale.set(width, height, depth)

      // Đặt lại vị trí khối sao cho tâm kho ở gốc toạ độ
      cube.position.set(
        baseX - centerX,
        baseY + index * height - centerY,
        baseZ - centerZ
      )

      scene.add(cube)
    })
  })

  const animate = () => {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
  }

  animate()

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth * 0.7 / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth * 0.7, window.innerHeight)
  })
})
</script>

<style scoped>
.container {
  display: flex;
  height: 100vh;
}

.scene-container {
  width: 70%;
  height: 100%;
}

.table-container {
  width: 30%;
  padding: 20px;
  overflow-y: auto;
  background-color: #f9f9f9;
}

.inventory-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

.inventory-table th,
.inventory-table td {
  border: 1px solid #ccc;
  padding: 6px 8px;
  text-align: left;
}

.inventory-table th {
  background-color: #eee;
}
</style>
