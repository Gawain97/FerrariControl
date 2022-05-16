<template>
  <div class="Controls">
    <button style="float: left" @click="createCube">创建一个cube</button>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
// import Stats from "three/examples/jsm/libs/stats.module.js";
import * as dat from "dat.gui";
import earth from "../imgs/earth_atmos_2048.jpg";
import sun from "../imgs/j025.jpg";
import earth_normal from "../imgs/earth_normal_2048.jpg";
import moon from "../imgs/moon_1024.jpg";
export default {
  name: "ControlsTest",
  data() {
    return {
      scene: new THREE.Scene(),
    };
  },
  mounted() {
    this.init();
    // this.createGroundGrid()
  },
  methods: {
    init() {
      // this.scene = new THREE.Scene();
      console.log(this.scene);

      const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        2000
      );
      camera.position.set(0, 500, 500);
      const loader = new THREE.TextureLoader();
      var earth1 = loader.load(earth);
      var earth2 = loader.load(earth_normal);
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      // renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      const gui = new dat.GUI();
      var geometry = new THREE.SphereGeometry(40, 32, 32);
      var material = new THREE.MeshPhongMaterial({
        // color: 0x0000ff,
        map: earth1,
        normalMap: earth2,
      });
      var mesh = new THREE.Mesh(geometry, material);
      // mesh.position.set(0,0,0)
      this.scene.add(mesh);

      // 设置产生投影的网格模型
      mesh.castShadow = true;

      //创建一个平面几何体作为投影面
      var planeGeometry = new THREE.PlaneGeometry(500, 500);
      var planeMaterial = new THREE.MeshPhongMaterial({
        color: 0x999999,
      });
      // 平面网格模型作为投影面
      var planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
      this.scene.add(planeMesh); //网格模型添加到场景中
      planeMesh.rotateX(-Math.PI / 2); //旋转网格模型
      // planeMesh.position.y = -50; //设置网格模型y坐标
      // 设置接收阴影的投影面
      planeMesh.receiveShadow = true;

      // 方向光
      var directionalLight = new THREE.DirectionalLight(0xffffff, 1);


      // 设置光源位置
      directionalLight.position.set(20, 10, 1);
      this.scene.add(directionalLight);
      // scene.add(DirectionalLightHelper);
      const ambientLight = new THREE.AmbientLight(0xcccccc);
      this.scene.add(ambientLight);
      // 设置用于计算阴影的光源对象
      directionalLight.castShadow = true;
      // 设置计算阴影的区域，最好刚好紧密包围在对象周围
      gui.add(directionalLight.shadow.camera, "near", 0, 1).step(0.005);
      gui.add(directionalLight.shadow.camera, "far", 0, 500).step(1);
      gui.add(directionalLight.shadow.camera, "left", -500, 500).step(1);
      gui.add(directionalLight.shadow.camera, "right", -500, 500).step(1);
      gui.add(directionalLight.shadow.camera, "top", -500, 500).step(1);
      gui.add(directionalLight.shadow.camera, "bottom", -500, 500).step(1);

      // 计算阴影的区域过大：模糊  过小：看不到或显示不完整
      directionalLight.shadow.camera.near = 0.5;
      directionalLight.shadow.camera.far = 500;
      directionalLight.shadow.camera.left = -100;
      directionalLight.shadow.camera.right = 100;
      directionalLight.shadow.camera.top = 100;
      directionalLight.shadow.camera.bottom = -100;
      // 设置mapSize属性可以使阴影更清晰，不那么模糊
      // directionalLight.shadow.mapSize.set(1024,1024)s
      console.log(directionalLight.shadow.camera);
      document
        .getElementsByClassName("Controls")[0]
        .appendChild(renderer.domElement);

      var axisHelper = new THREE.AxesHelper(100, 100, 100);
      this.scene.add(axisHelper);

      const controls = new OrbitControls(camera, renderer.domElement);
      const clock = new THREE.Clock();

      function render() {
        const time = clock.getElapsedTime();

        requestAnimationFrame(render);
        controls.update();
        console.log(this.scene);

        // renderer.render(this.scene, camera); //执行渲染操作
      }
      function onWindowResize() {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      }

      window.addEventListener("resize", onWindowResize);
      render();
    },
    createCube() {
      console.log("创建cube");
      const cubeG = new THREE.BoxGeometry(20, 20, 20);
      const cubeM = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      const cube = new THREE.Mesh(cubeG, cubeM);
      cube.position.set(50, 0, 0);
      this.scene.add(cube);
    },
  },
};
</script>

<style>
/* canvas{
    background-image: url("../imgs/bk.jpg");
    background-size: cover;

  } */
</style>