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
import { onMounted, reactive, ref, onUnmounted, nextTick } from 'vue'

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

// 颜色数组
const colors = [
  'red',
  'blue',
  'green',
  'orange',
  'purple',
  'gray'
]

// 汽车组件材质
const glassMaterialList = [
  { name: '常规', value: 1 },
  { name: '透亮', value: 0.2 }
]

// clear coat层的粗糙度
const bodyMaterialList = [
  { name: '磨砂', value: 1 },
  { name: '冰晶', value: 0 }
]

// 汽车颜色组件选项集合
const carColorList = [
  { name: '车身', mark: 'bodyMaterial' },
  { name: '引擎盖', mark: 'hoodMaterial' },
  { name: '前脸材质', mark: 'frontMaterial' },
  { name: '轮毂', mark: 'wheelsMaterial' },
  { name: '挡风玻璃', mark: 'glassMaterial' }
]

// 汽车材质组件选项集合
const carMaterialList = [
   { name: '全车', mark: 'bodyMaterial' },
   { name: '挡风玻璃', mark: 'glassMaterial' }
]

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


// 创建 车身 材质
const bodyMaterial = new THREE.MeshPhysicalMaterial( {
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0
} )

// 创建 前脸 材质
const frontMaterial = new THREE.MeshPhysicalMaterial( {
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0
} )

// 创建 引擎盖 材质
const hoodMaterial = new THREE.MeshPhysicalMaterial( {
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0
} )

  // 创建 挡风玻璃 材质
const glassMaterial = new THREE.MeshPhysicalMaterial( {
  color: 0xffffff,
  metalness: 0,
  roughness: 0.1,
  transmission: 1,
  transparent: true,
  opacity: 1
} )

// 创建 轮毂 材质
const wheelsMaterial = new THREE.MeshPhysicalMaterial( {
  color: 0xff0000,
  metalness: 1,
  roughness: 0.1
} )

// 点击颜色时修改对应汽车组件颜色
let selectColor = (index, mark) => {
  switch (mark) {
    case 'bodyMaterial':
      bodyMaterial.color.set(colors[index])
      break;
    case 'frontMaterial':
      frontMaterial.color.set(colors[index])
      break;
    case 'hoodMaterial':
      hoodMaterial.color.set(colors[index])
      break;
    case 'glassMaterial':
      glassMaterial.color.set(colors[index])
      break;
    case 'wheelsMaterial':
      wheelsMaterial.color.set(colors[index])
      break;
  }
}

// 点击材质修改对应汽车组件材质
let selectMaterials = (index, mark) => {
  switch (mark) {
    case 'bodyMaterial':
      bodyMaterial.clearcoatRoughness = bodyMaterialList[index].value
      frontMaterial.clearcoatRoughness = bodyMaterialList[index].value
      hoodMaterial.clearcoatRoughness = bodyMaterialList[index].value
      break;
    case 'glassMaterial':
      glassMaterial.opacity = glassMaterialList[index].value
      break;
  }
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

  // 初始化汽车组件
  let wheels = []
  let carBody, frontCar, hoodCar, glassCar

  gltfLoader.load(
    '/model/bmw01.glb',
    (gltf) => {
      const model = gltf.scene
      // model.scale.set(1.5,1.5,1.5)
      model.traverse(child => {
        /* if(child.isMesh) {
          console.log(child.name)
        } */

        // 判断是否是轮毂
        if(child.isMesh && child.name.includes('轮毂')){
          child.material = wheelsMaterial;
          wheels.push(child)
        }

        // 判断是否是车身
        if(child.isMesh && child.name.includes('Mesh002')){
          carBody = child
          carBody.material = bodyMaterial
        }

        // 判断是否是前脸
        if(child.isMesh && child.name.includes('前脸')){
          frontCar = child
          frontCar.material = frontMaterial
        }

        // 判断是否是引擎盖
        if(child.isMesh && child.name.includes('引擎盖_1')){
          hoodCar = child
          hoodCar.material = hoodMaterial
        }

        // 判断是否是挡风玻璃
        if(child.isMesh && child.name.includes('挡风玻璃')){
          glassCar = child
          glassCar.material = glassMaterial
        }
      })
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
    <!-- 场景 -->
    <div class='canvas' ref='canvas'></div>

    <!-- 选配面板 -->
    <div class='home-container'>

      <div class='home-container-title'>
        <h2>汽车展示与选配</h2>
      </div>

      <div class='select' v-for='(card,index) in carColorList.splice(0,4)' :key='index'>
        <!-- 选项卡标题 -->
        <h3 class='select-title'>{{ card.name }}选择</h3>
        
        <div class='selectNo'>
          <!-- 颜色选项 -->
            <div class='select-item' v-for='(item, index) in colors'
            :key='index' @click='selectColor(index, card.mark)'
            >
              <div class='select-item-color' :style="{ background: item }"></div>
            </div>
        </div>
      </div>

      <div class='select' v-for='(card,index) in carMaterialList' :key='index'>
        <!-- 选项卡标题 -->
        <h3 class='select-title'>{{ card.name }}选择</h3>
        
        <div class='selectNo' v-if="card.name === '全车'">
          <!-- 材质选项 全车 -->
            <div class='select-item' v-for='(gItem, gIndex) in bodyMaterialList'
            :key='index' @click='selectMaterials(gIndex, card.mark)'
            >
              <div class='select-item-text'> {{ gItem.name }} </div>
            </div>
        </div>

        <div class='selectNo' v-if="card.name === '挡风玻璃'">
          <!-- 材质选项 挡风玻璃 -->
            <div class='select-item' v-for='(bItem, bIndex) in glassMaterialList'
            :key='index' @click='selectMaterials(bIndex, card.mark)'
            >
              <div class='select-item-text'> {{ bItem.name }} </div>
            </div>
        </div>
      </div>


    </div>
  </div>
</template>

<style scoped lang='less'>
.home {
  width: 100vw;
  height: 100vh;

  .home-container {
    position: fixed;
    top: 0;
    right: 20px;

    .select {
      display: flex;
      flex-direction: column;
      justify-content: space-between;

      .select-title {
        display: flex;
      }

      .selectNo {
        display: flex;

        .select-item {
          

          .select-item-color, .select-item-text {
            flex-direction: row;
            border: 1px solid #eee;
            border-radius: 5px;
            margin: 2px;
            display: inline-block;
            cursor: pointer;
          }

          .select-item-color{
            width: 20px;
            height: 20px;
          }

          .select-item-text {
            width: 70px;
            height: 20px;
            font-size: 1px;
            text-align:center;
            line-height: 20px;
          }
        }
      }
    }
  }
}
</style>
