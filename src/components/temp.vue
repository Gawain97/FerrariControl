<template>
  <div class="Controls">
    <button @click="createPlane">创建一个星球</button>
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
      arr:[],
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
      this.checkCube();

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
      const cube1Color = {
        color: 0xffff00,
      };
      const bkColor = {
        color: 0x000000,
      };

      // console.log('clock3',this.clock3)
      const f1 = this.gui.addFolder("1号球体");
      const f2 = this.gui.addFolder("2号球体");
      const f3 = this.gui.addFolder("3号球体");
      // console.log(this.box1)
      // f1.add(this.box1, "scale", 1, 3).name("放大倍数");

      f1.add(this.group1.rotation, "x", -1, 1).name("旋转角度X");
      f1.add(this.group1.rotation, "z", -1, 1).name("旋转角度Z");
      f1.add(this.box1.material, "wireframe").name("1号球体材质");

      // console.log(this.control)
      f2.add(this.control, "speed", 0, 5).name("旋转速度");
      f2.add(this.box2.material, "wireframe").name("2号球体材质");

      // console.log(cube1Color.color)
      f2.addColor(cube1Color, "color").onChange(() => {
        this.box2Material.color = new THREE.Color(cube1Color.color);
        // console.log(this.box2Material)
      });
      f2.add(this.control, "stop1").name("停止/开始");

      f3.add(this.control, "speed2", 0, 20).name("旋转速度");
      f3.add(this.box3.material, "wireframe").name("1号球体材质");
      f3.add(this.control, "stop2").name("停止/开始");

      this.gui.add(this.control, "r1", 50, 100).name("1-2旋转半径");
      this.gui.add(this.control, "r2", 10, 20).name("2-3旋转半径");
      this.gui.add(this.ambientLight, "intensity", 0, 1).name("环境光强度");
      this.gui.add(this.directionalLight, "intensity", 0, 1).name("平行光强度");
      this.gui.add(this.parameters, "radius", 1, 30).name("球半径");

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
    // 添加几何体
    addBox() {
      this.group1 = new THREE.Group();
      this.group2 = new THREE.Group();

      // var box1 = new THREE.BoxGeometry(15, 15, 15);
      // 中心大球
      const loader = new THREE.TextureLoader();
      const sun1 = loader.load(sun);
      const mercury1 = loader.load(mercury);

      const earth1 = loader.load(earth);
      const earth2 = loader.load(earth_normal);
      const moon1 = loader.load(moon);

      const Gargantua = loader.load(gargantua);
      const cubeG = new THREE.SphereGeometry(30, 32, 32);
      const cubeM = new THREE.MeshLambertMaterial({
        map: mercury1,
      });
      this.cube = new THREE.Mesh(cubeG, cubeM);
      this.cube.position.set(150, 10, 0);
      this.cube.name='cube'

      this.cube.castShadow = true;
      // cube.receiveShadow=true

      this.scene.add(this.cube);

      this.box1geometry = new THREE.SphereGeometry(20, 32, 32);
      this.box1 = new THREE.Mesh(
        this.box1geometry,
        new THREE.MeshPhongMaterial({ map: sun1 })
      );
      this.group1.position.y = 20;
      // this.group1.castShadow=true
      this.box1.castShadow = true;
      this.box1.receiveShadow = true;

      console.log(this.group1);
      console.log(this.box1);

      this.group1.add(this.box1);

      // 2号球体
      var box2 = new THREE.SphereGeometry(10);
      this.box2Material = new THREE.MeshPhongMaterial({
        map: earth1,
        normalMap: earth2,
      });
      this.box2 = new THREE.Mesh(box2, this.box2Material);
      this.group2.add(this.box2);
      this.box2.castShadow = true;

      // this.group1.add(this.box2)

      var box3 = new THREE.SphereGeometry(2);
      this.box3 = new THREE.Mesh(
        box3,
        new THREE.MeshStandardMaterial({ map: moon1 })
      );
      this.box3.castShadow = true;
      this.group2.add(this.box3);
      this.group1.add(this.group2);

      this.scene.add(this.group1);

      const planeG = new THREE.PlaneGeometry(1000, 1000);
      const planeM = new THREE.MeshLambertMaterial({ map: Gargantua }); //,side:THREE.DoubleSide
      const plane = new THREE.Mesh(planeG, planeM);
      plane.position.y = -20;
      plane.rotation.x = -Math.PI / 2;
      plane.name = "plane";
      plane.castShadow = true;
      plane.receiveShadow = true;
      this.scene.add(plane);
    },
    // 动画
    animate() {
      this.box1geometry = new THREE.SphereGeometry(this.parameters.radius);
      // const clockll=this.clock
      // console.log(clockll)
      if (this.clock) {
        this.time = this.clock.getElapsedTime();
      }
      if (this.clock2) {
        this.time2 = this.clock2.getElapsedTime();
      }
      // if (this.composer) {
      //   // console.log(this.composer)
      //   this.composer.renderer.render(this.scene,this.camera);
      // }
      this.box1.rotation.y += 0.005;
      this.box2.rotation.y += 0.01;
      this.box3.rotation.y += 0.01;
      this.cube.rotation.y += 0.005;
      this.box2.position.x =
        Math.sin(this.time * this.control.speed) * this.control.r1;
      this.box2.position.z =
        Math.cos(this.time * this.control.speed) * this.control.r1;
      this.box3.position.x =
        this.box2.position.x +
        Math.sin(this.time2 * this.control.speed2) * this.control.r2;
      this.box3.position.z =
        this.box2.position.z +
        Math.cos(this.time2 * this.control.speed2) * this.control.r2;

      this.arr.forEach((item,index)=>{
        if(item.position.x>1000){
          item.position.x=0
        }
        item.rotation.y+=0.01
        item.position.x+=1
        // item.position.x=Math.sin(this.time) * (index+1)*100;
        // item.position.z=Math.cos(this.time) *(index+1)*100;
        

      })
      // this.box3.rotation.y += 0.001;

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
        2000
      );
      this.camera.position.set(200, 200, 200);
      this.camera.lookAt(this.scene.position);
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
      // this.renderer.setClearColor(0xcccccc);
      document
        .getElementsByClassName("Controls")[0]
        .appendChild(this.renderer.domElement);

      // document.body.appendChild(this.renderer.domElement);
    },
    // 鼠标控制
    initOrbitControls() {
      const controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.minDistance = 50;
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
    // 选中物体
    checkCube() {
      this.raycaster = new THREE.Raycaster();
      console.log(this.raycaster);
      this.mouse = new THREE.Vector2();
      this.renderer.domElement.addEventListener("mousedown", (event) => {
        this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
        console.log("点击鼠标", this.mouse);
        this.raycaster.setFromCamera(this.mouse, this.camera);

        var intersects = this.raycaster.intersectObjects(this.scene.children);
        console.log(intersects);
        if (intersects.length > 0) {
          if (intersects[0].object.name === "plane") {
            return;
          }
          console.log("intersects", intersects);
          var obj = intersects[0].object;
          // this.outlineObj(intersects)
          this.copyCube(obj);
          if (obj.material.wireframe) {
            obj.material.wireframe = false;
          }
          // else {
            // obj.material.wireframe = true;
          // }
          
          obj.material.needsUpdate = true;
        }
          console.log(this.scene.children)
          
          this.arr=this.scene.children.filter(item=>item.name==='cube')
          console.log(this.arr)

      });
    },
    copyCube(old) {
      // console.log('obj',old)

      const newG = old.geometry;
      const newM = old.material;
      console.log(newG, newM);
      var newobj = new THREE.Mesh(newG, newM);

      newobj.position.x = old.position.x + 100;
      newobj.position.y = old.position.y ;
      newobj.name=old.name


      this.scene.add(newobj);
      console.log(newobj);
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
canvas {
  background-image: url("../imgs/bk.jpg");
  background-size: cover;
}
button {
  position: absolute;
  margin-top: 50px;
}
</style>
