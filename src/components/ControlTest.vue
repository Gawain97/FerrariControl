<template>
  <div class="Controls"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import Stats from "three/examples/jsm/libs/stats.module.js";
import * as dat from "dat.gui";
export default {
  name: "ControlsTest",
  data() {
    return {
      gui: new dat.GUI(),
      ellipseRotation: Math.PI / 2,
      clock: new THREE.Clock(),
      clock2: new THREE.Clock(),

      clock1:0,
 
      flag: true,
      flag2: true,

      //this.positionX=this.group2.position.x,this.positionZ=this.group2.position.z,// positionX:0,// positionZ:0,
      control:{
        r1: 50,
        r2: 10,
        speed: 0.2,
        speed2: 4,
        stop1: () => {
        if (this.flag) {

          // console.log(this.group2.position.x, this.group2.position.z);
          this.control.speed = 0;
          this.clock=0
          this.flag = !this.flag;
          console.log('停止1',this.control.speed2,this.clock2,this.flag2)
        } else {
          this.flag = !this.flag;

          this.control.speed = 0.2;
          this.clock=new THREE.Clock()
          console.log('开始1',this.control.speed2,this.clock2,this.flag2)

        }
        },
        stop2: () => {
        if (this.flag2) {
   
          this.control.speed2 = 0;
          this.clock2=0


          this.flag2 = !this.flag2;

        } else {
          this.flag2 = !this.flag2;
          this.control.speed2 = 4;
          this.clock2=new THREE.Clock()

        }
        }

      },
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

      this.axiesHelper();

      this.iniStats();

      this.addBox();

      this.animate();
      this.initGui();
    },
    initGui() {
      const cube1Color={
        color:0xffff00
      }
      const bkColor={
        color:0x000000
      }

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
      f2.add(this.control, "speed",0,5).name("旋转速度");
      f2.add(this.box2.material, "wireframe").name("2号球体材质");

      // console.log(cube1Color.color)
      f2.addColor(cube1Color, "color").onChange(()=>{
        this.box2Material.color=new THREE.Color(cube1Color.color)
        // console.log(this.box2Material)
      })
      f2.add(this.control, "stop1").name("停止/开始");


      f3.add(this.control, "speed2",0,20).name("旋转速度");
      f3.add(this.box3.material, "wireframe").name("1号球体材质");
      f3.add(this.control, "stop2").name("停止/开始");





      this.gui.add(this.control, "r1",50,100).name("1-2旋转半径");
      this.gui.add(this.control, "r2",10,20).name("2-3旋转半径");
      this.gui.add(this.ambientLight, "intensity",0,1).name("环境光强度");
      this.gui.add(this.directionalLight, "intensity",0,1).name("平行光强度");  
      console.log(this.renderer)

      this.gui.addColor(bkColor, "color").onChange(()=>{
        this.renderer.setClearColor(bkColor.color)
      })



      // this.gui.add()
      console.log(this.gui)

    },
    // 添加几何体
    addBox() {
      this.group1 = new THREE.Group();
      this.group2 = new THREE.Group();

      // var box1 = new THREE.BoxGeometry(15, 15, 15);
      // 中心大球
      var box1 = new THREE.SphereGeometry(15);
      this.box1 = new THREE.Mesh(
        box1,
        new THREE.MeshNormalMaterial({ wireframe: true })
      );
      this.group1.add(this.box1)
      // 2号球体
      var box2 = new THREE.SphereGeometry(5);
      this.box2Material=new THREE.MeshPhongMaterial({ wireframe: true ,color:0xff00ff})
      this.box2 = new THREE.Mesh(
        box2,
        this.box2Material
      );
      this.group2.add(this.box2)
      // this.group1.add(this.box2)


      var box3 = new THREE.SphereGeometry(2);
      this.box3 = new THREE.Mesh(
        box3,
        new THREE.MeshNormalMaterial({ wireframe: true })
      );
      this.group2.add(this.box3)
      this.group1.add(this.group2)

      this.scene.add(this.group1);
      // this.scene.add(this.group2);

  // console.log(this.box2.position)
      // this.scene.add(this.box2);
      // this.scene.add(this.box3);



    },

    // 动画
    animate() {
      // const clockll=this.clock
      // console.log(clockll)
      if(this.clock){

        this.time = this.clock.getElapsedTime();
      }
      if(this.clock2){
        this.time2 = this.clock2.getElapsedTime();
      }

      this.box1.rotation.y += 0.005;
      this.box2.rotation.y += 0.005;
      this.box3.rotation.y += 0.005;

      this.box2.position.x = Math.sin(this.time * this.control.speed) * this.control.r1;
      this.box2.position.z = Math.cos(this.time * this.control.speed) * this.control.r1;
      this.box3.position.x = this.box2.position.x+Math.sin(this.time2 * this.control.speed2) * this.control.r2
      this.box3.position.z = this.box2.position.z+Math.cos(this.time2 * this.control.speed2) *  this.control.r2

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
    },
    // 相机
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        2000
      );
      this.camera.position.set(100, 100, 100);
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
       this.ambientLight = new THREE.AmbientLight(0xcccccc, 0.5);
      this.scene.add(this.ambientLight);

      this.directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
      this.directionalLight.position.set(-1, 1, 1);
      this.scene.add(this.directionalLight);
    },
    // 渲染器
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setPixelRatio(window.devicePixelRatio);
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
      controls.maxDistance = 500;
    },
    // 帧率
    iniStats() {
      this.stats = new Stats();
      document
        .getElementsByClassName("Controls")[0]
        .appendChild(this.stats.dom);
      // document.body.appendChild(this.stats.dom);
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
</style>