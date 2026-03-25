<template>
    <div ref="container" class="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import { markRaw } from 'vue';
import { OrbitControls} from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass';


export default {
    name: 'ThreeScene',
    props: {
        width: { type: Number, default: window.innerWidth},
        height: {type: Number,default: window.innerHeight - 100}
    },
    data(){
        return {
            clock: new THREE.Clock(),
            objects: []
        };
    },
    mounted(){
        this.scene = null;
        this.camera = null;
        this.renderer = null;
        this.controls = null;
        this.composer = null;
        this.init();
        this.animate();
        window.addEventListener('resize',this.onResize);
    },
    beforeDestroy(){
        window.removeEventListener('resize',this.onResize);
        if (this.composer) {
            this.composer.dispose();
        }
        this.renderer.dispose();
    },
    methods: {
        init(){
            //1.创建场景
            this.scene = markRaw(new THREE.Scene());
            this.scene.background = new THREE.Color(0x050505);
            this.scene.fog = new THREE.Fog(0x050505,10,50);

            //2.创建相机
            this.camera = markRaw(new THREE.PerspectiveCamera(
                60,
                this.width / this.height,
                0.1,
                1000
            ));
            this.camera.position.set(0,5,15);

            //3.创建渲染器
            /**
             * @type{THREE.WebGLRenderer}
             */
            this.renderer = markRaw(new THREE.WebGLRenderer({
                antialias: true,
                alpha: true
            }));
            this.renderer.setSize(this.width,this.height);
            this.renderer.setPixelRatio(window.devicePixelRatio);
            this.renderer.shadowMap.enabled = true;
            this.renderer.shadowMap.type = THREE.PCF;
            
            this.$refs.container.appendChild(this.renderer.domElement);

            //4.添加轨道控制器
            this.controls= markRaw(new OrbitControls(this.camera,this.renderer.domElement));
            this.controls.enableDamping = true;
            this.controls.dampingFactor = 0.05;
            this.controls.minDistance = 5;
            
            //5.设置光照
            this.setupLights();

            //6.添加3D物品
            this.createObjects();

            //7.后期处理效果
            this.setupPostProcessing();
        },

        setupLights(){
            //环境光
            const ambientLight = new THREE.AmbientLight(0x404040,0.8);
            this.scene.add(ambientLight);

            //方向光(模拟太阳)
            const directionalLight = new THREE.DirectionalLight(0x404040,0.8);
            this.scene.add(directionalLight);
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 2048;
            directionalLight.shadow.mapSize.height = 2048;

            //点光源
            const pointLight = new THREE.PointLight(0x0088ff,2,100);
            pointLight.position.set(5,10,5);
            this.scene.add(pointLight);
        },

      createObjects(){
          //示例1：炫酷的几何体组合
        const geometry = new THREE.IcosahedronGeometry(3,1);
        const material = new THREE.MeshPhongMaterial({
          color: 0x00a8ff,
          shininess: 100,
          transparent: true,
          opacity: 0.9,
          wireframe: false
        });

        const mesh = markRaw(new THREE.Mesh(geometry, material));
        mesh.position.set(0,3,0);
        mesh.castShadow = true;
        this.scene.add(mesh);
        this.objects.push(mesh);

        //示例2：粒子系统
        this.createParticleSystem();

        //示例3：加载3D模型（如果需要）
        //this.loadGLTFModel();

        //示例4：地面
        const groundGeometry = new THREE.PlaneGeometry(50,50);
        const groundMaterial = new THREE.MeshPhongMaterial({
          color: 0x333333,
          side: THREE.DoubleSide
        });
        const ground = markRaw(new THREE.Mesh(groundGeometry, groundMaterial));
        ground.rotation.x = -Math.PI / 2;
        ground.rotation.y = -5;
        ground.receiveShadow = true;
        this.scene.add(ground);
      },

      createParticleSystem(){
          const particleCount = 2000;
          const particles = new THREE.BufferGeometry();

          const positions = new Float32Array(particleCount * 3);
          const colors = new Float32Array(particleCount * 3);

          for (let i = 0; i < particleCount * 3; i+=3) {
            //位置
            positions[i] = (Math.random() - 0.5)*20;
            positions[i+1] = (Math.random() - 0.5)*20;
            positions[i+2] = (Math.random() - 0.5)*20;

            //颜色
            colors[i] = Math.random();
            colors[i+1] = Math.random();
            colors[i+2] = Math.random();
          }

          particles.setAttribute('position', new THREE.BufferAttribute(positions,3));
          particles.setAttribute('color',new THREE.BufferAttribute(colors,3));

          const particleMaterial = new THREE.PointsMaterial({
            size: 0.1,
            vertexColors: true,
            transparent: true,
            opacity: 0.8
          });

          const particleSystem = markRaw(new THREE.Points(particles,particleMaterial));
          this.scene.add(particleSystem);
          this.objects.push(particleSystem);
      },

      async loadGLTFModel(){
          try {
            const loader = new GLTFLoader();
            const gltf = await loader.loadAsync('/models/your-model.gltf');

            gltf.scene.traverse((child) => {
              if(child.isMesh){
                child.castShadow = true;
                child.receiveShadow = true;
              }
            });

            gltf.scene.position.set(0, 0, 0);
            gltf.scene.scale.set(0.5,0.5,0.5);
            this.scene.add(gltf.scene);
            this.objects.push(gltf.scene);
          }catch (error){
            console.error("加载模型失败：",error);
          }
      },

      setupPostProcessing(){
          //创建后期处理通道
        this.composer = markRaw(new EffectComposer(this.renderer));
        this.composer.addPass(new RenderPass(this.scene,this.camera));

        //添加辉光效果
        const bloomPass = new UnrealBloomPass(
            new THREE.Vector2(window.innerWidth,window.innerHeight),
            1.5,
            0.4,
            0.85
        );
        this.composer.addPass(bloomPass);
      },

      animate(){
          requestAnimationFrame(this.animate);

          const delta = this.clock.getDelta();

          //更新物体动画
        this.objects.forEach((obj,index) => {
          if (obj.isMesh){
            obj.rotation.x += delta * 0.5;
            obj.rotation.y += delta * 0.5;

            //添加呼吸效果
            const scale = 1+ Math.sin(Date.now() * 0.001+index)* 0.1;
            obj.scale.setScalar(scale);
          }
        });

        //更新控制器
        if (this.controls){
          this.controls.update();
        }

        //渲染
        if (this.composer){
          this.composer.render();
        }else {
          this.renderer.render(this.scene,this.camera);
        }
      },

      onResize(){
          this.width = window.innerWidth;
          this.height = window.innerHeight - 100;
          this.camera.aspect = this.width / this.height;
          this.camera.updateProjectionMatrix();
          this.renderer.setSize(this.width, this.height);

          if (this.composer){
            this.composer.setSize(this.width, this.height);
          }
      },

      toggleEffect() {
          // 切换效果的逻辑
          if (this.objects.length > 0) {
              const mesh = this.objects[0];
              if (mesh.material) {
                  mesh.material.wireframe = !mesh.material.wireframe;
              }
          }
      },

      changeColor() {
          // 改变颜色的逻辑
          if (this.objects.length > 0) {
              const mesh = this.objects[0];
              if (mesh.material) {
                  // 生成随机颜色
                  const randomColor = new THREE.Color(Math.random(), Math.random(), Math.random());
                  mesh.material.color = randomColor;
              }
          }
      }
    }

};
</script>

<style scoped>
.three-container{
  width: 100%;
  height: calc(100vh - 100px);
  background: linear-gradient(135deg,#667eea 0%,#764ba2 100%);
  position: relative;
  overflow: hidden;
}
</style>