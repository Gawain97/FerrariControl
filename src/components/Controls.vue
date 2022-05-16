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
      speed: 0.2,
      r: 50,
      flag: true,
      stop: () => {
        if (this.flag) {
          const X=this.group2.position.x
          const Y=this.group2.position.z
          // console.log(this.group2.position.x, this.group2.position.z);
          this.speed = 0;
          this.clock=0
          console.log(this.group2.position.x, this.group2.position.z);

          this.flag = !this.flag;
          console.log("停止", this.speed);
        } else {
          this.flag = !this.flag;

          this.speed = 0.2;
          this.clock=new THREE.Clock()
          console.log("开始", this.speed);
        }
      }, //this.positionX=this.group2.position.x,this.positionZ=this.group2.position.z,
      // positionX:0,
      // positionZ:0,
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
      const control = {
        r: this.r,
        speed: this.speed,
        stop: this.stop,

      };
      const f1 = this.gui.addFolder("1号球体");
      const f2 = this.gui.addFolder("2号球体");
      const f3 = this.gui.addFolder("3号球体");

      f1.add(this.group1.rotation, "x", -1, 1).name("1号球体旋转角度X");
      f1.add(this.group1.rotation, "z", -1, 1).name("1号球体旋转角度Z");

      f1.add(this.box1.material, "wireframe").name("1号球体材质");
      f2.add(control, "stop").name("停止/开始");
      f2.add(this.ellipse.scale, "x",1,2).name("X轴缩放");
      f2.add(this.ellipse.scale, "y",1,2).name("Y轴缩放");


      console.log(this.ellipse)

      // this.gui.add(this.box1.rotation, "y").name("大球旋转");
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
      const ambientLight = new THREE.AmbientLight(0xcccccc, 0.4);
      this.scene.add(ambientLight);

      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
      directionalLight.position.set(-1, 1, 1);
      this.scene.add(directionalLight);
    },
    // 渲染器
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setClearColor(0xcccccc);
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
    // 添加几何体
    addBox() {
      console.log("1111");
      this.group1 = new THREE.Group();
      this.group2 = new THREE.Group();

      // var box1 = new THREE.BoxGeometry(15, 15, 15);
      // 中心大球
      var box1 = new THREE.SphereGeometry(15);
      this.box1 = new THREE.Mesh(
        box1,
        new THREE.MeshNormalMaterial({ wireframe: true })
      );
      // 圆环1
      this.curve = new THREE.EllipseCurve(
        0,
        0, // ax, aY
        this.r,
        this.r, // xRadius, yRadius
        0,
        2 * Math.PI, // aStartAngle, aEndAngle
        false, // aClockwise
        0 // aRotation
      );
      // 圆环2
      var curve2 = new THREE.EllipseCurve(
        0,
        0, // ax, aY
        15,
        15, // xRadius, yRadius
        0,
        2 * Math.PI, // aStartAngle, aEndAngle
        false, // aClockwise
        0 // aRotation
      );

      var points = this.curve.getPoints(50);
      var points2 = curve2.getPoints(50);

      var geometry = new THREE.BufferGeometry().setFromPoints(points);
      console.log('geometry',geometry)
      var geometry2 = new THREE.BufferGeometry().setFromPoints(points2);

      var material = new THREE.LineBasicMaterial({ color: 0xff0000 });

      // 大圆环
      this.ellipse = new THREE.Line(geometry, material);
      this.ellipse.rotateX(this.ellipseRotation);

      // 加入group1
      this.group1.add(this.box1);
      this.group1.add(this.ellipse);

      // 2号球体
      var box2 = new THREE.SphereGeometry(5);
      this.box2 = new THREE.Mesh(
        box2,
        new THREE.MeshNormalMaterial({ wireframe: true })
      );

      this.group2.add(this.box2);

      var box3 = new THREE.SphereGeometry(1);
      this.box3 = new THREE.Mesh(
        box3,
        new THREE.MeshNormalMaterial({ wireframe: true })
      );
      this.box3.position.x = 10;
      // this.scene.add(this.box3);

      this.group2.add(this.box3);

      this.group1.add(this.group2);
      this.group2.position.x = 50;

      this.ellipse2 = new THREE.Line(geometry2, material);
      console.log(this.ellipse2)
      this.ellipse2.rotateX(this.ellipseRotation);
      this.group2.add(this.ellipse2);
      this.scene.add(this.group1);
      // this.scene.add(this.group2)

      // console.log(this.group1)
    },

    // 动画
    animate() {
      // this.group1.rotation.y += 0.005;
      // const clock =new THREE.Clock()
      if(this.clock){
        this.time = this.clock.getElapsedTime();
      }

      // console.log(Math.sin(time))

      // this.group2.rotation.y += 0.01;
      this.box1.rotation.y += 0.005;
      this.group2.position.x = Math.sin(this.time * this.speed) * this.r;
      this.group2.position.z = Math.cos(this.time * this.speed) * this.r;

      this.box3.rotation.y += 0.001;

      requestAnimationFrame(this.animate);
      this.stats.update();
      // this.initGui();
      this.render();
    },
    // 坐标系
    axiesHelper() {
      var helper = new THREE.AxesHelper(50);
      this.scene.add(helper);
    },
    // 渲染
    render() {
      this.renderer.render(this.scene, this.camera);
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