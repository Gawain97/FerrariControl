<template>
  <div id="container"></div>
</template>
 
<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import Stats from "three/examples/jsm/libs/stats.module.js";

export default {
  name: "ThreeTest",
  components: {},

  data() {
    return {};
  },

  mounted() {
    // three 一些参数定义（这样可以必要的避免卡顿）
    console.log(this);
    this.scene;
    this.camera;
    this.renderer;
    this.stats;
    // this.box1;

    this.init();
  },
  methods: {
    // 初始化
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
    },

    initScene() {
      this.scene = new THREE.Scene();
      // console.log('scene',this.scene.position)
    },

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

    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      window.addEventListener("resize", this.onWindowResize);
    },

    initLight() {
      const ambientLight = new THREE.AmbientLight(0xcccccc, 0.4);
      this.scene.add(ambientLight);

      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
      directionalLight.position.set(-1, 1, 1);
      this.scene.add(directionalLight);
    },

    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setClearColor(0xcccccc);
      document.body.appendChild(this.renderer.domElement);
    },

    initOrbitControls() {
      const controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.minDistance = 50;
      controls.maxDistance = 500;
    },

    iniStats() {
      this.stats = new Stats();
      document.body.appendChild(this.stats.dom);
    },

    addBox() {
      this.group1 = new THREE.Group();
      this.group2 = new THREE.Group();

      // var box1 = new THREE.BoxGeometry(15, 15, 15);
      var box1 = new THREE.SphereGeometry( 15);
      
      this.box1 = new THREE.Mesh(box1, new THREE.MeshNormalMaterial({wireframe:true}));

      var curve = new THREE.EllipseCurve(
        0,
        0, // ax, aY
        50,
        50, // xRadius, yRadius
        0,
        2 * Math.PI, // aStartAngle, aEndAngle
        false, // aClockwise
        90 // aRotation
      );
      var curve2 = new THREE.EllipseCurve(
        0,
        0, // ax, aY
        10,
        10, // xRadius, yRadius
        0,
        2 * Math.PI, // aStartAngle, aEndAngle
        false, // aClockwise
        90 // aRotation
      );

      var points = curve.getPoints(50);
      var points2 = curve2.getPoints(50);

      var geometry = new THREE.BufferGeometry().setFromPoints(points);
      var geometry2 = new THREE.BufferGeometry().setFromPoints(points2);


      var material = new THREE.LineBasicMaterial({ color: 0xff0000 });

      // Create the final object to add to the scene
      this.ellipse = new THREE.Line(geometry, material);
      this.ellipse.rotateX(Math.PI / 2);
      // this.scene.add(this.box1);
      this.group1.add(this.box1);
      this.scene.add(this.ellipse);

      // var box2 = new THREE.BoxGeometry(5, 5, 5);
      var box2 = new THREE.SphereGeometry( 5 );
      this.box2 = new THREE.Mesh(box2, new THREE.MeshNormalMaterial({wireframe:true}));
      // this.box2.position.x = -20;
      // this.scene.add(this.box2);
      this.group2.add(this.box2);
      // this.group1.add(this.box2)

      // var box3 = new THREE.BoxGeometry(1, 1, 1);
      var box3 = new THREE.SphereGeometry( 1 );

      this.box3 = new THREE.Mesh(box3, new THREE.MeshNormalMaterial({wireframe:true}));
      this.box3.position.x = 10;
      // this.scene.add(this.box3);

      this.group2.add(this.box3);

      this.group1.add(this.group2);
      this.group2.position.x = 50;

      this.ellipse2 = new THREE.Line(geometry2, material);
      this.ellipse2.rotateX(Math.PI / 2);
      this.group2.add(this.ellipse2);
      this.scene.add(this.group1);
      // this.scene.add(this.group2)

      // console.log(this.group1)
    },

    moveBox() {
      this.box.position.set(10, 0, 10);
    },

    animate() {
      this.group1.rotation.y += 0.005;
      this.group2.rotation.y += 0.01;
      this.box1.rotation.y += 0.005;
      this.box2.rotation.y += 0.002;
      this.box3.rotation.y += 0.001;

      requestAnimationFrame(this.animate);
      this.stats.update();
      this.render();
    },

    axiesHelper() {
      var helper = new THREE.AxesHelper(50);
      this.scene.add(helper);
    },

    render() {
      this.renderer.render(this.scene, this.camera);
    },
  },

  // beforeDestroy 废弃，使用 beforeUnmount
  beforeUnmount() {
    // 相关参数置空
    this.scene = null;
    this.camera = null;
    this.renderer = null;
    this.stats = null;
    this.box = null;
  },
};
</script>
 
<style>
#app {
  text-align: center;
  height: 100%;
}
</style>