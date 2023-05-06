<script setup>
// 引入threejs
import * as THREE from 'three'

// 引入轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

// 引入GLTF加载器
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"

// DRACOLoader
import { DRACOLoader  } from 'three/examples/jsm/loaders/DRACOLoader'

// 引入组合式API
import { onMounted, reactive, ref, onUnmounted } from 'vue'

// 画布节点
const canvas = ref(null)

// 声明控制器
let control

// 创建场景
const scene = new THREE.Scene()

// 创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)

// 初始化相机位置
camera.position.set(0, 2, 6)

// 将相机添加到场景
scene.add(camera)

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true })
renderer.setSize(window.innerWidth, window.innerHeight)

// 坐标格辅助对象. 坐标格实际上是2维线数组.
const gridHelper = new THREE.GridHelper(10, 10)
gridHelper.material.opacity = 0.2
gridHelper.material.transparent = true
scene.add(gridHelper)

// 添加环境光
/* const ambientLight = new THREE.AmbientLight({ color: 0xffffff, intensity: 1})
scene.add(ambientLight) */

// 添加平行光
const light1 = new THREE.DirectionalLight(0xffffff, 0.5)
light1.position.z = 10
scene.add(light1)

const light2 = new THREE.DirectionalLight(0xffffff, 1)
light1.position.z = -10
scene.add(light2)

const light3 = new THREE.DirectionalLight(0xffffff, 1)
light1.position.x = 10
scene.add(light3)

const light4 = new THREE.DirectionalLight(0xffffff, 1)
light1.position.x = -10
scene.add(light4)

const light5 = new THREE.DirectionalLight(0xffffff, 1)
light1.position.y = 10
scene.add(light5)

const light6 = new THREE.DirectionalLight(0xffffff, 1)
light1.position.set(5, 10, 0)
scene.add(light6)

const light7 = new THREE.DirectionalLight(0xffffff, 0.5)
light1.position.set(0, 10, 5)
scene.add(light7)

const light8 = new THREE.DirectionalLight(0xffffff, 0.5)
light1.position.set(0, 10, -5)
scene.add(light8)

const light9 = new THREE.DirectionalLight(0xffffff, 0.5)
light1.position.set(-5, 10, 0)
scene.add(light9)

// 渲染函数
const renderFn = () => {
  renderer.render(scene, camera)
  requestAnimationFrame(renderFn)
  control && control.update()
}

// 侦听修改渲染器、相机宽高大小
const handlResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix ()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// 组件挂载完毕
onMounted(() => {
  // 把渲染器插入到画布节点中
  canvas.value.appendChild( renderer.domElement )

  // 清除默认背景颜色
  renderer.setClearColor('#000')

  // 设置场景背景和环境
  scene.background = new THREE.Color('#ccc')
  scene.environment = new THREE.Color('#ccc')

  // 创建控制器 control已经在全局中声明了
  control = new OrbitControls(camera, canvas.value)
  control.update()

  // 创建汽车模型
  const gltfLoader = new GLTFLoader()
  const dracoLoader = new DRACOLoader()

  dracoLoader.setDecoderPath( '/draco/gltf/' )
  gltfLoader.setDRACOLoader( dracoLoader )

  gltfLoader.load(
    '/model/bmw01.glb',
    (gltf) => {
      const model = gltf.scene
      model.scale.set(1.5,1.5,1.5)
      scene.add(model)
    }
  )

  // 将侦听渲染器、相机宽高大小改变函数加到window
  window.addEventListener('resize', handlResize)

  // renderFn
  renderFn()
})

// 组件卸载后
onUnmounted(() => {
  window.removeEventListener('resize', handlResize)
})

</script>

<template>
  <div class='home'>
    <div class='canvas' ref='canvas'></div>
  </div>
</template>

<style scoped>
.home {
  width: 100vw;
  height: 100vh;
}
</style>
