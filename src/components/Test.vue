<template>
  <div>
    <div id="container"></div>
    <div class="controls-box">
      <section>
        <el-row>
          <div v-for="(item,key) in properties" :key="key">
            <div>
              <el-col :span="8">
                <span class="vertice-span">{{item.name}}</span>
              </el-col>
              <el-col :span="13">
                <el-slider v-model="item.value" :min="item.min" :max="item.max" :step="item.step" :format-tooltip="formatTooltip"></el-slider>
              </el-col>
              <el-col :span="3">
                <span class="vertice-span">{{item.value}}</span>
              </el-col>
            </div>
          </div>
        </el-row>
      </section>
    </div>
  </div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { SceneUtils } from 'three/examples/jsm/utils/SceneUtils.js'
export default {
  data() {
    return {
      properties: {
        radius: {
          name: 'radius',
          value: 8,
          min: 0,
          max: 40,
          step: 1
        },
        widthSegments: {
          name: 'widthSeg',
          value: 10,
          min: 0,
          max: 20,
          step: 1
        },
        heightSegments: {
          name: 'heightSeg',
          value: 10,
          min: 0,
          max: 20,
          step: 1
        },
        phiStart: {
          name: 'phiStart',
          value: 0,
          min: 0,
          max: Math.PI * 2,
          step: 0.1
        },
        phiLength: {
          name: 'phiLength',
          value: Math.PI * 2,
          min: 0,
          max: Math.PI * 2,
          step: 0.1
        },
        thetaStart: {
          name: 'thetaStart',
          value: 0,
          min: 0,
          max: Math.PI * 2,
          step: 0.1
        },
        thetaLength: {
          name: 'thetaLength',
          value: Math.PI,
          min: 0,
          max: Math.PI * 2,
          step: 0.1
        }
      },
      mesh: null,
      camera: null,
      scene: null,
      renderer: null,
      controls: null
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    formatTooltip(val) {
      return val
    },
    // 初始化
    init() {
      this.createScene() // 创建场景
      this.createMesh() // 创建网格模型
      this.createLight() // 创建光源
      this.createCamera() // 创建相机
      this.createRender() // 创建渲染器
      this.createControls() // 创建控件对象
      this.render() // 渲染
    },
    // 创建场景
    createScene() {
      this.scene = new THREE.Scene()
    },
    // 创建网格模型
    createMesh() {
      //创建三维球体
      const geom = new THREE.SphereGeometry(
        this.properties.radius.value,
        this.properties.widthSegments.value,
        this.properties.heightSegments.value,
        this.properties.phiStart.value,
        this.properties.phiLength.value,
        this.properties.thetaStart.value,
        this.properties.thetaLength.value
      )
      // 创建材质
      const meshMaterial = new THREE.MeshNormalMaterial({
        side: THREE.DoubleSide
      })
      const wireFrameMat = new THREE.MeshBasicMaterial({ wireframe: true })

      // 添加组合材质
      this.mesh = SceneUtils.createMultiMaterialObject(geom, [
        meshMaterial,
        wireFrameMat
      ])

      // 网格对象添加到场景中
      this.scene.add(this.mesh)
    },

    // 创建光源
    createLight() {
      // 环境光
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.1) // 创建环境光
      this.scene.add(ambientLight) // 将环境光添加到场景

      const spotLight = new THREE.SpotLight(0xffffff) // 创建聚光灯
      spotLight.position.set(-40, 60, -10)
      spotLight.castShadow = true
      this.scene.add(spotLight)
    },
    // 创建相机
    createCamera() {
      const element = document.getElementById('container')
      const width = element.clientWidth // 窗口宽度
      const height = element.clientHeight // 窗口高度
      const k = width / height // 窗口宽高比
      // PerspectiveCamera( fov, aspect, near, far )
      this.camera = new THREE.PerspectiveCamera(35, k, 0.1, 1000)
      this.camera.position.set(-80, 60, 40) // 设置相机位置

      this.camera.lookAt(new THREE.Vector3(10, 0, 0)) // 设置相机方向
      this.scene.add(this.camera)
    },
    // 创建渲染器
    createRender() {
      const element = document.getElementById('container')
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
      this.renderer.setSize(element.clientWidth, element.clientHeight) // 设置渲染区域尺寸
      this.renderer.shadowMap.enabled = true // 显示阴影
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
      this.renderer.setClearColor(0x3f3f3f, 1) // 设置背景颜色
      element.appendChild(this.renderer.domElement)
    },

    // 更新属性
    updateFun() {
      const tempRotationY = this.mesh.rotation.y
      this.scene.remove(this.mesh)
      this.createMesh()
      this.mesh.rotation.y += tempRotationY + 0.01
    },
    render() {
      this.updateFun()
      this.renderer.render(this.scene, this.camera)
      requestAnimationFrame(this.render)
    },
    // 创建控件对象
    createControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    }
  }
}
</script>
<style>
#container {
  position: absolute;
  width: 100%;
  height: 100%;
}
.controls-box {
  position: absolute;
  right: 5px;
  top: 5px;
  width: 300px;
  padding: 10px;
  background-color: #fff;
  border: 1px solid #c3c3c3;
}
.vertice-span {
  line-height: 38px;
  padding: 0 2px 0 10px;
}
</style>

