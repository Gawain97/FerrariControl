<template>
  <div>
    <div id="container">
      <!-- <div id="info">
        <span class="colorPicker"
          ><input
            id="body-color"
            type="color"
            value="#ffffff"
          /><br />车身</span
        >
        <span class="colorPicker"
          ><input
            id="details-color"
            type="color"
            value="#ffffff"
          /><br />Details</span
        >
        <span class="colorPicker"
          ><input
            id="glass-color"
            type="color"
            value="#ffffff"
          /><br />Glass</span
        >
        <span class="colorPicker"
          ><input
            id="wheels-color"
            type="color"
            value="#ffffff"
          /><br />车轮</span
        >
        <span class="colorPicker"
          ><input
            id="test-color"
            type="color"
            value="#ffffff"
          /><br />测试</span
        >
      </div> -->
    </div>
  </div>
</template>

<script>
import * as THREE from "three";

import Stats from "three/examples/jsm/libs/stats.module.js";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import * as dat from "dat.gui";
export default {
  data() {
    return {
      wheels: [],
      gui: new dat.GUI(),
      cheshen: {},
      // carModel:{}
    };
  },
  mounted() {
    this.wheels = [];
    this.init();
    this.initCamera();
    // renderer
    this.initRenderer();

    // OrbitControls
    this.initOrbitControls();

    // onWindowResize
    this.onWindowResize();

    // this.axiesHelper();

    this.iniStats();
    this.animate();
    this.initLight();
  },
  methods: {
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(
        40,
        window.innerWidth / window.innerHeight,
        0.1,
        100
      );
      this.camera.position.set(4.25, 1.4, -4.5);
    },
    // 窗口监听
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      window.addEventListener("resize", this.onWindowResize);
    },
    // 渲染器
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setAnimationLoop(this.render);
      this.renderer.outputEncoding = THREE.sRGBEncoding;
      this.renderer.toneMapping = THREE.ACESFilmicToneMapping;
      this.renderer.toneMappingExposure = 0.85;
      this.renderer.shadowMap.enabled = true;
      // this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      console.log("renderer", this.renderer);
      this.renderer.setClearColor(0xcccccc);
      document
        .getElementById("container")
        .appendChild(this.renderer.domElement);

      // document.body.appendChild(this.renderer.domElement);
    },
    // 帧率
    iniStats() {
      this.stats = new Stats();
      document.getElementById("container").appendChild(this.stats.dom);
      // document.body.appendChild(this.stats.dom);
    },
    initGui() {
      const Carcolor = {
        color: 0xffff00,
      };
      let material = {
        metalness: 0.5,
        roughness: 0.5,
        clearcoat: 0.5,
        clearcoatRoughness: 0.5,
        specular: 0x111111,
      };
      const body = this.gui.addFolder("车身");
      let cheshen = this.carModels.getObjectByName("body");
      console.log("车身材质", cheshen.material.type);
      let bodyMaterial = body.addFolder("配置");

      const wheels = this.gui.addFolder("车轮");
      let glass = this.carModels.getObjectByName("glass");
      let glassMaterial = wheels.addFolder("配置");

      let MeshStandardMaterial = new THREE.MeshStandardMaterial({
        color: 0xff0000,
        emissive: 0x0,
        metalness: 0.5,
        roughness: 0.5,
      });
      let MeshPhongMaterial = new THREE.MeshPhongMaterial({
        // emissive: 0x00ff00,
        color: 0xff00ff,
        emissive: 0x0,
        shininess: 30,
        specular: 0x111111,
      });
      let MeshPhysicalMaterial = new THREE.MeshPhysicalMaterial({
        color: 0x0000ff,
        emissive: 0x0,
        metalness: 0.5,
        roughness: 0.5,
        clearcoat: 0.5,
        clearcoatRoughness: 0.5,
        transmission: 1.0,
      });

      // 材质可见
      let visible = true;
      body
        .add(cheshen.material, "visible")
        .onChange(() => {
          cheshen.material.visible = !visible;
          visible = !visible;
        })
        .name("显示");
      // console.log("bodyMaterial", bodyMaterial);
      wheels
        .add(glass.material, "visible")
        .onChange(() => {
          glass.material.visible = !visible;
          visible = !visible;
        })
        .name("显示");
      // create(glassMaterial,glass)
      function create(fname, mtl) {
        fname
          .add(mtl.material, "type", [
            "MeshPhysicalMaterial",
            "MeshStandardMaterial",
            "MeshPhongMaterial",
          ])
          .onChange((e) => {
            console.log(e);
            if (e === "MeshPhysicalMaterial") {
              if (mtl.material.type === "MeshStandardMaterial") {
                console.log("before1", mtl.material.type);
              } else if (mtl.material.type === "MeshPhongMaterial") {
                console.log("before2", mtl.material.type);
                fname.remove(this.shininess);
                fname.remove(this.specular);
                this.metalness = fname
                  .add(material, "metalness", 0, 1)
                  .onChange((e) => {
                    mtl.material.metalness = e;
                  })
                  .name("metalness");
                this.roughness = fname
                  .add(material, "roughness", 0, 1)
                  .onChange((e) => {
                    mtl.material.roughness = e;
                  })
                  .name("roughness");
              }
              mtl.material = MeshPhysicalMaterial;
              // bodyMaterial.remove(this.shininess);
              // bodyMaterial.remove(this.specular);
              // bodyMaterial.remove(this.metalness);
              // bodyMaterial.remove(this.roughness);

              this.clearcoat = fname
                .add(material, "clearcoat", 0, 1)
                .onChange((e) => {
                  mtl.material.clearcoat = e;
                })
                .name("clearcoat");
              this.clearcoatRoughness = fname
                .add(material, "clearcoatRoughness", 0, 1)
                .onChange((e) => {
                  mtl.material.clearcoatRoughness = e;
                })
                .name("clearcoatRoughness");
            } else if (e === "MeshStandardMaterial") {
              console.log("before", mtl.material);
              if (mtl.material.type === "MeshPhysicalMaterial") {
                console.log("before3", mtl.material.type);
                fname.remove(this.clearcoat);
                fname.remove(this.clearcoatRoughness);
              } else if (mtl.material.type === "MeshPhongMaterial") {
                console.log("before4", mtl.material.type);
                fname.remove(this.shininess);
                fname.remove(this.specular);
                this.metalness = fname
                  .add(material, "metalness", 0, 1)
                  .onChange((e) => {
                    mtl.material.metalness = e;
                  })
                  .name("metalness");
                this.roughness = fname
                  .add(material, "roughness", 0, 1)
                  .onChange((e) => {
                    mtl.material.roughness = e;
                  })
                  .name("roughness");
                // bodyMaterial.hide();
              }
              mtl.material = MeshStandardMaterial;

              // bodyMaterial.remove(this.metalness);
              // bodyMaterial.remove(this.roughness);
            } else if (e === "MeshPhongMaterial") {
              if (mtl.material.type === "MeshPhysicalMaterial") {
                console.log("before5", mtl.material.type);
                fname.remove(this.clearcoat);
                fname.remove(this.clearcoatRoughness);
              } else if (mtl.material.type === "MeshStandardMaterial") {
                console.log("before6", mtl.material.type);
              }
              mtl.material = MeshPhongMaterial;
              fname.remove(this.metalness);
              fname.remove(this.roughness);
              this.shininess = fname
                .add(mtl.material, "shininess", 1, 100)
                .name("高亮程度");
              this.specular = fname
                .addColor(material, "specular")
                .onChange(() => {
                  mtl.material.specular = new THREE.Color(
                    material.specular
                  );
                })
                .name("高光颜色");
            }
          });
        // console.log("ches", cheshen.material);

        //材质颜色
        fname
          .addColor(Carcolor, "color")
          .onChange(() => {
            mtl.material.color = new THREE.Color(Carcolor.color);
          })
          .name("车身颜色");

        fname
          .addColor(Carcolor, "color")
          .onChange(() => {
            mtl.material.emissive = new THREE.Color(Carcolor.color);
          })
          .name("放射光");
        this.metalness = fname
          .add(material, "metalness", 0, 1)
          .onChange((e) => {
            mtl.material.metalness = e;
          })
          .name("metalness");
        this.roughness = fname
          .add(material, "roughness", 0, 1)
          .onChange((e) => {
            mtl.material.roughness = e;
          })
          .name("roughness");
      }


      // <---------------------------------------->
      //材质选择
      bodyMaterial
        .add(cheshen.material, "type", [
          "MeshPhysicalMaterial",
          "MeshStandardMaterial",
          "MeshPhongMaterial",
        ])
        .onChange((e) => {
          console.log(e);
          if (e === "MeshPhysicalMaterial") {
            if (cheshen.material.type === "MeshStandardMaterial") {
              console.log("before1", cheshen.material.type);
            } else if (cheshen.material.type === "MeshPhongMaterial") {
              console.log("before2", cheshen.material.type);
              bodyMaterial.remove(this.shininess);
              bodyMaterial.remove(this.specular);
              this.metalness = bodyMaterial
                .add(material, "metalness", 0, 1)
                .onChange((e) => {
                  cheshen.material.metalness = e;
                })
                .name("metalness");
              this.roughness = bodyMaterial
                .add(material, "roughness", 0, 1)
                .onChange((e) => {
                  cheshen.material.roughness = e;
                })
                .name("roughness");
            }
            cheshen.material = MeshPhysicalMaterial;
            // bodyMaterial.remove(this.shininess);
            // bodyMaterial.remove(this.specular);
            // bodyMaterial.remove(this.metalness);
            // bodyMaterial.remove(this.roughness);

            this.clearcoat = bodyMaterial
              .add(material, "clearcoat", 0, 1)
              .onChange((e) => {
                cheshen.material.clearcoat = e;
              })
              .name("clearcoat");
            this.clearcoatRoughness = bodyMaterial
              .add(material, "clearcoatRoughness", 0, 1)
              .onChange((e) => {
                cheshen.material.clearcoatRoughness = e;
              })
              .name("clearcoatRoughness");
          } else if (e === "MeshStandardMaterial") {
            console.log("before", cheshen.material);
            if (cheshen.material.type === "MeshPhysicalMaterial") {
              console.log("before3", cheshen.material.type);
              bodyMaterial.remove(this.clearcoat);
              bodyMaterial.remove(this.clearcoatRoughness);
            } else if (cheshen.material.type === "MeshPhongMaterial") {
              console.log("before4", cheshen.material.type);
              bodyMaterial.remove(this.shininess);
              bodyMaterial.remove(this.specular);
              this.metalness = bodyMaterial
                .add(material, "metalness", 0, 1)
                .onChange((e) => {
                  cheshen.material.metalness = e;
                })
                .name("metalness");
              this.roughness = bodyMaterial
                .add(material, "roughness", 0, 1)
                .onChange((e) => {
                  cheshen.material.roughness = e;
                })
                .name("roughness");
              // bodyMaterial.hide();
            }
            cheshen.material = MeshStandardMaterial;

            // bodyMaterial.remove(this.metalness);
            // bodyMaterial.remove(this.roughness);
          } else if (e === "MeshPhongMaterial") {
            if (cheshen.material.type === "MeshPhysicalMaterial") {
              console.log("before5", cheshen.material.type);
              bodyMaterial.remove(this.clearcoat);
              bodyMaterial.remove(this.clearcoatRoughness);
            } else if (cheshen.material.type === "MeshStandardMaterial") {
              console.log("before6", cheshen.material.type);
            }
            cheshen.material = MeshPhongMaterial;
            bodyMaterial.remove(this.metalness);
            bodyMaterial.remove(this.roughness);
            this.shininess = bodyMaterial
              .add(cheshen.material, "shininess", 1, 100)
              .name("高亮程度");
            this.specular = bodyMaterial
              .addColor(material, "specular")
              .onChange(() => {
                cheshen.material.specular = new THREE.Color(material.specular);
              })
              .name("高光颜色");
          }
        });
      // console.log("ches", cheshen.material);

      //材质颜色
      bodyMaterial
        .addColor(Carcolor, "color")
        .onChange(() => {
          cheshen.material.color = new THREE.Color(Carcolor.color);
        })
        .name("车身颜色");

      bodyMaterial
        .addColor(Carcolor, "color")
        .onChange(() => {
          cheshen.material.emissive = new THREE.Color(Carcolor.color);
        })
        .name("放射光");
      this.metalness = bodyMaterial
        .add(material, "metalness", 0, 1)
        .onChange((e) => {
          cheshen.material.metalness = e;
        })
        .name("metalness");
      this.roughness = bodyMaterial
        .add(material, "roughness", 0, 1)
        .onChange((e) => {
          cheshen.material.roughness = e;
        })
        .name("roughness");
    },
    init() {
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0x333333);
      this.scene.environment = new RGBELoader().load(
        "models/ferrari/venice_sunset_1k.hdr"
      );
      this.scene.environment.mapping = THREE.EquirectangularReflectionMapping;
      this.scene.fog = new THREE.Fog(0x333333, 10, 15);

      this.grid = new THREE.GridHelper(20, 40, 0xffffff, 0xffffff);
      this.grid.material.opacity = 0.2;
      this.grid.material.depthWrite = false;
      this.grid.material.transparent = true;
      this.scene.add(this.grid);

      // materials

      const bodyMaterial = new THREE.MeshStandardMaterial({
        color: 0xffffff,
        // metalness: 1.0,
        // roughness: 0.5,
        // clearcoat: 1.0,
        // clearcoatRoughness: 0.03,
        // sheen: 0.5,
      });

      const detailsMaterial = new THREE.MeshStandardMaterial({
        color: 0xffffff,
        metalness: 1.0,
        roughness: 0.5,
      });

      const glassMaterial = new THREE.MeshPhysicalMaterial({
        color: 0xffffff,
        metalness: 0.25,
        roughness: 0,
        transmission: 1.0,
      });
      const wheelsMaterial = new THREE.MeshPhysicalMaterial({
        color: 0xffffff,
        metalness: 0.25,
        roughness: 0,
        transmission: 1.0,
      });
      const testMaterial = new THREE.MeshStandardMaterial({
        color: 0xffffff,
        metalness: 0.25,
        roughness: 0,
      });

      // Car

      const shadow = new THREE.TextureLoader().load(
        "models/ferrari/ferrari_ao.png"
      );

      const dracoLoader = new DRACOLoader();
      dracoLoader.setDecoderPath("models/ferrari/draco/");

      const loader = new GLTFLoader();
      loader.setDRACOLoader(dracoLoader);
      // let that = this;

      loader.load("models/ferrari/ferrari.glb", (gltf) => {
        // console.log("before",that.carModel)
        this.carModels = gltf.scene.children[0];
        const carModel = gltf.scene.children[0];

        console.log("falali", carModel);
        console.log("falalis", this.carModels);

        // console.log('body',carModel.getObjectByName("body"))

        // carModel.getObjectByName("trim").material = detailsMaterial; //座椅线0
        // carModel.getObjectByName("lights").material = testMaterial;//车灯1
        // carModel.getObjectByName("leds").material = testMaterial;//led小车灯2
        // carModel.getObjectByName("metal").material = testMaterial;//车窗金属3
        // carModel.getObjectByName("plastic_gray").material = testMaterial;//仪表盘4
        // carModel.getObjectByName("grills").material = testMaterial;//散热孔5
        // carModel.getObjectByName("lights_red").material = testMaterial; //安全带插槽6
        // carModel.getObjectByName("glass").material = glassMaterial;//车灯罩7
        // carModel.getObjectByName("carpet").material = testMaterial; //车内底盘8
        // carModel.getObjectByName("leather").material = testMaterial; //座椅和中控9
        // carModel.getObjectByName("carbon_fibre_trim").material = testMaterial; //中控碳纤维装饰10
        // carModel.getObjectByName("carbon_fibre").material = testMaterial; //换挡碳纤维拨片11
        // carModel.getObjectByName("chrome").material = testMaterial; //车标字母12
        // carModel.getObjectByName("brakes").material = testMaterial; //尾灯13
        // carModel.getObjectByName("blue").material = testMaterial; //未知14
        // carModel.getObjectByName("wipers").material = testMaterial; //雨刮器15
        // carModel.getObjectByName("yellow_trim").material = testMaterial; //车标16
        // carModel.getObjectByName("interior_dark").material = testMaterial; //车底盘17
        // carModel.getObjectByName("interior_light").material = testMaterial; //挡风玻璃框18
        carModel.getObjectByName("body").material = bodyMaterial; //车身19
        console.log("carmodel", carModel.getObjectByName("body"));

        //
        // that.cheshen=carModel.getObjectByName("body")

        {
          // 一个车轮
          carModel.getObjectByName("rim_fl").material = detailsMaterial; //轮圈
          carModel.getObjectByName("tire").material = wheelsMaterial; //轮胎
          carModel.getObjectByName("brake").material = wheelsMaterial; //刹车片
          carModel.getObjectByName("nuts").material = wheelsMaterial; //螺母
          carModel.getObjectByName("wheel").material = wheelsMaterial; //轮毂内
          carModel.getObjectByName("centre").material = wheelsMaterial; //轮毂中间
        }
        {
          //     const steering_wheel=carModel.getObjectByName("steering_wheel")//方向盘
          //  steering_wheel.getObjectByName("steering_centre").material = detailsMaterial;//方向盘中心
          //  steering_wheel.getObjectByName("steering_column").material = detailsMaterial;//转向柱
          //  steering_wheel.getObjectByName("steering_metal").material = detailsMaterial;//转向金属
          //  steering_wheel.getObjectByName("steering_red_lights").material = detailsMaterial;//转向红灯
          //  steering_wheel.getObjectByName("steering_trim").material = detailsMaterial;//方向盘和U形饰件
          //  steering_wheel.getObjectByName("steering_leather").material = detailsMaterial;//转向皮革
          //  steering_wheel.getObjectByName("steering_carbon").material = detailsMaterial;//转向碳纤维
        }

        carModel.getObjectByName("rim_fr").material = detailsMaterial;
        carModel.getObjectByName("rim_rr").material = wheelsMaterial;
        carModel.getObjectByName("rim_rl").material = detailsMaterial;

        //test

        this.wheels.push(
          carModel.getObjectByName("wheel_fl"),
          carModel.getObjectByName("wheel_fr"),
          carModel.getObjectByName("wheel_rl"),
          carModel.getObjectByName("wheel_rr")
        );

        // shadow
        const mesh = new THREE.Mesh(
          new THREE.PlaneGeometry(0.655 * 4, 1.3 * 4),
          new THREE.MeshBasicMaterial({
            map: shadow,
            blending: THREE.MultiplyBlending,
            toneMapped: false,
            transparent: true,
          })
        );
        mesh.rotation.x = -Math.PI / 2;
        mesh.renderOrder = 2;
        carModel.add(mesh);

        this.scene.add(carModel);
        this.initGui();
      });
      // console.log("this.carmodel", this.carModels.getObjectByName("body"));
    },

    // 动画
    animate() {
      requestAnimationFrame(this.animate);
      // const time = -performance.now() / 1000;

      // for (let i = 0; i < this.wheels.length; i++) {
      //   this.wheels[i].rotation.x = time * Math.PI * 2;
      // }

      // this.grid.position.z = -time % 1;

      this.stats.update();
      // this.initGui();
      this.render();
    },
    render() {
      this.renderer.render(this.scene, this.camera);
    },
    // 鼠标控制
    initOrbitControls() {
      const controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.minDistance = 1;
      controls.maxDistance = 5000;
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

      this.scene.add(helper);
      // console.log('directionalLight',this.directionalLight)
    },
  },
};
</script>

<style>
.colorPicker {
  display: inline-block;
  margin: 0 10px;
  margin-left: 100px;
  color: white;
}
#info {
  position: absolute;
  top: 0px;
  width: 100%;
  padding: 10px;
  box-sizing: border-box;
  text-align: center;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
  user-select: none;
  /* pointer-events: none; */
  /* z-index: 1; TODO Solve this in HTML */
}
#info #body-color {
  z-index: 2;
}
</style>