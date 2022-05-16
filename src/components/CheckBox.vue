<template>
  <div class="Controls">
    <!-- <button @click="createPlane">创建一个星球</button> -->
    <div id="info">Description</div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import Stats from "three/examples/jsm/libs/stats.module.js";
import * as dat from "dat.gui";
import earth from "../imgs/earth_atmos_2048.jpg";
import sun from "../imgs/j025.jpg";
import earth_normal from "../imgs/earth_normal_2048.jpg";
import moon from "../imgs/moon_1024.jpg";
import mercury from "../imgs/Mercury.jpg";
import gargantua from "../imgs/Gargantua.jpg";
// import Saturn from "../imgs/Savue.png"
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
import { OutlinePass } from "three/examples/jsm/postprocessing/OutlinePass.js";
import { ShaderPass } from "three/examples/jsm/postprocessing/ShaderPass.js";
import { FXAAShader } from "three/examples/jsm/shaders/FXAAShader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader.js";
import { MTLLoader } from "three/examples/jsm/loaders/MTLLoader.js";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
export default {
  name: "ControlsTest",
  data() {
    return {
      imgList: [
        { id: 1, imgSrc: require("../imgs/Jupiter.jpg") },
        { id: 2, imgSrc: require("../imgs/Mercury.jpg") },
        { id: 3, imgSrc: require("../imgs/Saturn.png") },
        { id: 4, imgSrc: require("../imgs/Venus.jpg") },
        { id: 5, imgSrc: require("../imgs/j007.jpg") },
        { id: 6, imgSrc: require("../imgs/j008.jpg") },
        { id: 7, imgSrc: require("../imgs/j009.jpg") },
        { id: 8, imgSrc: require("../imgs/j011.jpg") },
        { id: 9, imgSrc: require("../imgs/j015.jpg") },
        { id: 10, imgSrc: require("../imgs/j017.jpg") },
        { id: 11, imgSrc: require("../imgs/j024.jpg") },
        { id: 12, imgSrc: require("../imgs/j027.jpg") },
      ],
      gui: new dat.GUI(),
      ellipseRotation: Math.PI / 2,
      clock: new THREE.Clock(),
      clock2: new THREE.Clock(),
      arr: [],
      clock1: 0,

      flag: true,
      flag2: true,

      //this.positionX=this.group2.position.x,this.positionZ=this.group2.position.z,// positionX:0,// positionZ:0,
      control: {
        r1: 50,
        r2: 10,
        speed: 0.2,
        speed2: 4,
        stop1: () => {
          if (this.flag) {
            this.control.speed = 0;
            this.clock = 0;
            this.flag = !this.flag;
            //   console.log('停止1',this.control.speed2,this.clock2,this.flag2)
          } else {
            this.flag = !this.flag;

            this.control.speed = 0.2;
            this.clock = new THREE.Clock();
            //   console.log('开始1',this.control.speed2,this.clock2,this.flag2)
          }
        },
        stop2: () => {
          if (this.flag2) {
            this.control.speed2 = 0;
            this.clock2 = 0;

            this.flag2 = !this.flag2;
          } else {
            this.flag2 = !this.flag2;
            this.control.speed2 = 4;
            this.clock2 = new THREE.Clock();
          }
        },
      },
      parameters: {
        radius: 20,
      },
      composer: null,
      outlinePass: null,
      renderPass: null,
    };
  },
  mounted() {
    // three 一些参数定义（这样可以必要的避免卡顿）
    console.log(this);
    this.scene;
    this.camera;
    this.renderer;
    this.stats;
    // this.box1;
    this.gui;
    this.init();
  },
  methods: {
    init() {
      // scene

      this.initScene();

      // camera
      this.initCamera();

      // light
      this.initLight();

      // renderer
      this.initRenderer();

      // OrbitControls
      this.initOrbitControls();

      // onWindowResize
      this.onWindowResize();

      // this.axiesHelper();

      this.iniStats();

      this.addBox();
      // this.checkCube();

      this.animate();
      this.initGui();
      // this.outlineObj();
      // this.inintLoader()
    },
    inintLoader() {
      this.loader = new THREE.TextureLoader();
      this.earth1 = this.loader.load(earth);
      console.log("earth", this.earth1);
    },
    initGui() {
      const bkColor = {
        color: 0x000000,
      };

      this.gui.add(this.ambientLight, "intensity", 0, 1).name("环境光强度");
      this.gui.add(this.directionalLight, "intensity", 0, 1).name("平行光强度");

      //   console.log(this.renderer)

      this.gui.addColor(bkColor, "color").onChange(() => {
        this.renderer.setClearColor(bkColor.color);
      });

      // this.gui.add()
      console.log(this.gui);
    },
    //高亮显示模型（呼吸灯）
    outlineObj(selectedObjects) {
      // 创建一个EffectComposer（效果组合器）对象，然后在该对象上添加后期处理通道。
      this.composer = new EffectComposer(this.renderer);
      // 新建一个场景通道  为了覆盖到原理来的场景上
      this.renderPass = new RenderPass(this.scene, this.camera);
      this.composer.addPass(this.renderPass);
      // 物体边缘发光通道
      this.outlinePass = new OutlinePass(
        new THREE.Vector2(window.innerWidth, window.innerHeight),
        this.scene,
        this.camera,
        selectedObjects
      );
      this.outlinePass.selectedObjects = selectedObjects;
      this.outlinePass.edgeStrength = 10.0; // 边框的亮度
      this.outlinePass.edgeGlow = 1; // 光晕[0,1]
      this.outlinePass.usePatternTexture = false; // 是否使用父级的材质
      this.outlinePass.edgeThickness = 1.0; // 边框宽度
      this.outlinePass.downSampleRatio = 1; // 边框弯曲度
      this.outlinePass.pulsePeriod = 5; // 呼吸闪烁的速度
      this.outlinePass.visibleEdgeColor.set(parseInt(0x00ff00)); // 呼吸显示的颜色
      this.outlinePass.hiddenEdgeColor = new THREE.Color(0, 0, 0); // 呼吸消失的颜色
      this.outlinePass.clear = true;
      this.composer.addPass(this.outlinePass);
      // 自定义的着色器通道 作为参数
      var effectFXAA = new ShaderPass(FXAAShader);
      effectFXAA.uniforms.resolution.value.set(
        1 / window.innerWidth,
        1 / window.innerHeight
      );
      effectFXAA.renderToScreen = true;
      this.composer.addPass(effectFXAA);
    },
    LoaderModels() {
      let that = this;

      // var loader =new FBXLoader()
      // loader.load('models/uv5.fbx',function(obj){
      //   console.log('obj',obj)
      //   that.scene.add(obj)

      // })
      var loader = new GLTFLoader();
      let mtlLoader = new MTLLoader();
      var objLoader = new OBJLoader();
      //小狗车
      // loader.load(
      //   // 模型在 /public/static/building/文件夹下

      //   "models/wagon-dog/scene.gltf",

      //   (gltf) => {
      //     console.log("sd", gltf);
      //     this.scene.add(gltf.scene);
      //   }
      // );

      loader.load(
        // 模型在 /public/static/building/文件夹下

        "models/a_windy_day/scene.gltf",

        (gltf) => {
          console.log("sd", gltf);
          this.scene.add(gltf.scene);
        }
      );

      // mtlLoader.load(
      //   "models/southern-district-police-station/source/model/model.mtl",
      //   function (materials) {
      //     materials.preload();
      //     var mat=materials
      //     objLoader.setMaterials(mat)
      //     objLoader.load(
      //       "models/southern-district-police-station/source/model/model.obj",
      //       function (obj) {
      //         obj.position.set(0, -20, 0); // 平移位置

      //         that.scene.add(obj);
      //       }
      //     );
      //   }
      // );
      // mtlLoader.load(
      //   "models/bugatti/bugatti.mtl",
      //   function (materials) {
      //     materials.preload();
      //     var mat = materials;
      //     objLoader.setMaterials(mat);
      //     objLoader.load(
      //       "models/bugatti/bugatti.obj",
      //       function (obj) {
      //         obj.position.set(0, -20, 0); // 平移位置

      //         that.scene.add(obj);
      //       }
      //     );
      //   }
      // );
      // objLoader.load(
      //   "models/southern-district-police-station/source/model/model.obj",
      //   (geometry) => {
      //     console.log("geometry", geometry);
      //     this.scene.add(geometry);
      //   }
      // );
    },
    // 添加几何体
    addBox() {
      this.LoaderModels();
      // const scene = new THREE.Scene();
      // {
      //   const color = 0xff0000; // white
      //   const near = 1;
      //   const far = 200;
      //   this.scene.fog = new THREE.Fog(color, near, far);
      // }
    },
    // 动画
    animate() {
      requestAnimationFrame(this.animate);
      this.stats.update();
      // this.initGui();
      this.render();
    },
    // 坐标系
    axiesHelper() {
      var helper = new THREE.AxesHelper(100);
      this.scene.add(helper);
    },
    // 渲染
    render() {
      this.renderer.render(this.scene, this.camera);
    },
    // 场景
    initScene() {
      this.scene = new THREE.Scene();
      console.log("chaangjing", this.scene);
    },
    // 相机
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        5000
      );
      this.camera.position.set(0, 0, 20);
      // this.camera.lookAt(this.scene.position);
    },
    // 窗口监听
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      window.addEventListener("resize", this.onWindowResize);
    },
    // 灯光
    initLight() {
      this.ambientLight = new THREE.AmbientLight(0xcccccc, 0.1);
      this.scene.add(this.ambientLight);
      // this.pointlight = new THREE.PointLight(0xffffff, 2);
      // this.pointlight.position.set(0, 35, 0);
      // this.pointlight.castShadow = true;
      // this.scene.add(this.pointlight);

      this.directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      this.directionalLight.position.set(100, 100, 100);
      this.scene.add(this.directionalLight);

      this.directionalLight.castShadow = true;
      this.directionalLight.shadow.camera.near = 0.5;
      this.directionalLight.shadow.camera.far = 500;
      this.directionalLight.shadow.camera.left = -500;
      this.directionalLight.shadow.camera.right = 500;
      this.directionalLight.shadow.camera.top = 500;
      this.directionalLight.shadow.camera.bottom = -500;
      this.directionalLight.shadow.mapSize.set(1024, 1024);

      var helper = new THREE.DirectionalLightHelper(this.directionalLight, 50);

      // this.scene.add(helper);
      // console.log('directionalLight',this.directionalLight)
    },
    // 渲染器
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      // this.renderer.shadowMapEnabled =true
      this.renderer.shadowMap.enabled = true;
      // this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      console.log("renderer", this.renderer);
      this.renderer.setClearColor(0xcccccc);
      document
        .getElementsByClassName("Controls")[0]
        .appendChild(this.renderer.domElement);

      // document.body.appendChild(this.renderer.domElement);
    },
    // 鼠标控制
    initOrbitControls() {
      const controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.minDistance = 1;
      controls.maxDistance = 5000;
    },
    // 帧率
    iniStats() {
      this.stats = new Stats();
      document
        .getElementsByClassName("Controls")[0]
        .appendChild(this.stats.dom);
      document.body.appendChild(this.stats.dom);
    },
    createPlane() {
      const loader = new THREE.TextureLoader();
      const index = Math.floor(Math.random() * 11);
      console.log(Math.floor(11.11));
      const material = loader.load(this.imgList[index].imgSrc);
      console.log(this.imgList[index].imgSrc);
      const tempG = new THREE.SphereGeometry(
        Math.abs(Math.random() - 0.5) * 30
      );
      const tempM = new THREE.MeshBasicMaterial({ map: material });
      const temp = new THREE.Mesh(tempG, tempM);
      temp.position.set(
        Math.random() * 100,
        Math.random() * 100,
        Math.random() * 100
      );
      // temp.rotation.y += Math.random();
      this.scene.add(temp);
      console.log("创建成功", this.scene);
    },
  },
  beforeUnmount() {
    // 相关参数置空
    this.scene = null;
    this.camera = null;
    this.renderer = null;
    this.stats = null;
    this.box = null;
    // delete this.gui;
  },
};
</script>

<style>
/* canvas {
  background-image: url("../imgs/bk.jpg");
  background-size: cover;
} */
button {
  position: absolute;
  margin-top: 50px;
}
/* #info {
  position: absolute;
  top: 10px;
  width: 100px;
  text-align: center;
  z-index: 100;
  display: block;
  margin-left: 50%;
  background-color: rgb(224, 187, 142);
  border: 5px solid white;
} */
</style>