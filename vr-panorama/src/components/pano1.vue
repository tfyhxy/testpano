<template>
  <div id="pano1">
    <!-- <div id="container"
      @mousedown.prevent="down($event)"
      @mouseup.prevent="up"
      @mousemove.prevent="move($event)"
      @mousewheel.prevent="wheel($event)" >
      
    </div> -->
      <!-- <div id="container"
        @mousedown.prevent="down($event)"
        @mouseup.prevent="up"
        @mousemove.prevent="move($event)" @click="change()"></div>
      </div> -->
    <div id="container"></div>
  </div>
</template>

<script>
import * as THREE from "three";
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
export default {
  name: "pano1",
  //数据来源
  data() {
    return {
      first: false,               //首次标志
      loading: true,              //加载标志
      model: 1,                   //当前场景下标（初始为1）
      modelName: 'theRestaurant', //当前场景模块名称
      index: 1,                   //当前场景轮播当前下标，用于轮播（初始为1）
      total: 6,                   //当前场景轮播下标最大值，用于轮播
      small: 1,                   //当前场景轮播下标最小值，用于轮播
      scene: null,                //场景对象
      camera: null,               //相机对象
      renderer: null,             //渲染对象
      controls: null,             //空间对象
      mesh: null,                 //网格对象
      panoramicRotation: true,    //全景是否自动旋转（默认旋转）
      clock: null,                //自动旋转对比时间
      timeCookie: 0,              //自动旋转记录时间
      FPS: 40,                    //自动旋转帧数（每秒）
      sound: false,               //背景音乐打开或关闭标志
      audio: null,                //背景音乐播放器
      screen: false,              //全屏打开或者关闭标志
      isMouseDown: false,
      object:[],
    };
  },

  // 构造函数
  created() {
    this.$nextTick(() => {
      this.defaultRun();
    })
  },

  // mounted() {
  //   //全景自适应屏幕大小
  //   let _this = this;
  //   window.onresize = function () {
  //     _this.selfAdaption();
  //   };
  //   this.initAudio();
  // },
  
  //普通方法
  methods: {
    //全景默认启动
    defaultRun() {
      if (!this.first) {
        this.initScene();
        this.initCamera();
        this.initPanorama();
        this.initlight();
        this.initobj();
        this.initRenderer();
        this.refresh();
        // this.initControls();

      }
    },

    //场景生成
    initScene() {
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color("#fff");
      this.scene.add( new THREE.AmbientLight( 0xffffff, 0.3 ) )
      const light = new THREE.DirectionalLight( 0xffffff, 0.35 );
				light.position.set( 1, 1, 1 ).normalize();
				this.scene.add( light );
    },

    //相机生成
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(90, window.innerWidth / window.innerHeight, 0.001, 10000);
      this.camera.zoom = 1;
      this.camera.updateProjectionMatrix();
      this.camera.position.set(0, 0, 0);
      // this.camera.lookAt(this.scene.position);
      // this.camera = new CinematicCamera( 60, window.innerWidth / window.innerHeight, 1, 1000 );
			// 	this.camera.setLens( 5 );
			// 	this.camera.position.set( 2, 1, 500 );
    },

    //生成全景
    initPanorama() {
        let textureLoader = new THREE.TextureLoader().load(require("../../public/static/panorama/8K Sample Square_1.jpg"), () => {
          this.loading = false;
          let box = new THREE.SphereGeometry(100, 100, 100);//创建全景球体
          let material = new THREE.MeshBasicMaterial({color: 0xffffff, side: THREE.BackSide,}); //全景球体样式
          this.mesh = new THREE.Mesh(box, material);
          this.mesh.material.map = textureLoader;
          // this.mesh.position= new THREE.Vector3(100,20,20)
          this.scene.add(this.mesh);
        });
    },

    initlight() {
       var light = new THREE.DirectionalLight(0xffffff); /*方向性光源*/
                //light.position.set(600, 1000, 800);
                // var light = new THREE.AmbientLight(0xffffff); //模拟漫反射光源
                light.position.set(0, 0, 0); 
                this.scene.add(light);
    },

    initobj() {
      for(let i=0;i<4;i++){
        const geometry = new THREE.BoxGeometry(1, 1, 1);
        this.object[i] = new THREE.Mesh( geometry, new THREE.MeshLambertMaterial( { color: Math.random() * 0xffffff } ) );

        this.object[i].position.x =  Math.random()*60;
        this.object[i].position.y = 0;
        this.object[i].position.z =  -Math.random()*60;

        this.scene.add( this.object[i] );
      }
    },

    //生成渲染
    initRenderer() {
      let container = document.getElementById("container");
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      container.appendChild(this.renderer.domElement);
    },

    //生成控件对象
    initControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);//创建控件对象
      this.controls.autoRotate = true
      this.controls.enablePan = false;//是否可平移
      this.controls.maxDistance = 10;//缩放上限
      this.controls.addEventListener('change', this.refresh);//监听鼠标、键盘事件
      console.log('tag')
    },
    
    //实时刷新渲染
    refresh() {
      // if (null == this.clock) this.clock = new THREE.Clock();
      // if (this.panoramicRotation && this.clock && this.mesh) {
      //   this.timeCookie = this.timeCookie + this.clock.getDelta();
      //   if (this.timeCookie > (1 / this.FPS)) {
      //     this.mesh.rotateY(0.002);
      //     this.timeCookie = 0;
      //   }
      // }
      this.renderer.render(this.scene, this.camera);
      // this.camera.updateProjectionMatrix();
      requestAnimationFrame(this.refresh);
    },
    
    change() {
      let a= new Date()
      while(this.camera.position.y<100){
        // setTimeout(()=>{this.camera.fov-=10},100)
        let b=new Date()
        this.camera.position.y+=(b-a)*0.1
        this.camera.updateProjectionMatrix();
        a=b
      }

      // setTimeout(() => {
      //   let textureLoader = new THREE.TextureLoader().load(require("../../public/static/panorama/huaweipanorama/PIC_20211014_154856_20211015092950.jpg"), () => {
      //   this.mesh.material.map = textureLoader;
      //   // this.camera.updateProjectionMatrix();
        
      // });
      // },10000)
    },
    //左右切换场景
    // onClickCarousel(tag) {
    //   this.index = ('left' === tag) ? (((this.index - 1) < this.small) ? this.total : this.index - 1) : (((this.index + 1) > this.total) ? this.small : this.index + 1);
    //   let textureLoader = new THREE.TextureLoader().load('./static/room/' + this.modelName + '/' + this.index + '.jpg', () => {
    //     this.mesh.material.map = textureLoader;
    //     this.camera.updateProjectionMatrix();
    //   });
    // },
    //场景切换按钮
    // onClickButton(path, model, modeTotal) {
    //   if (model !== this.model) {
    //     this.model = model;
    //     this.total = modeTotal;
    //     this.index = 1;
    //     this.modelName = path;
    //     let textureLoader = new THREE.TextureLoader().load('./static/room/' + path + '/1.jpg', () => {
    //       this.mesh.material.map = textureLoader;
    //       this.camera.updateProjectionMatrix();
    //     });
    //   }
    // },

    down(e) {
      this.mouseX = e.x;
      this.mouseY = e.y;
      this.isMouseDown = true;
    },

    up() {
      this.isMouseDown = false;
    },

    // move(e) {
    //   if (this.isMouseDown) {
    //     this.mesh.rotation.y += -0.005 * (e.x - this.mouseX);
    //     this.mesh.rotation.x += -0.005 * (e.y - this.mouseY);
    //     if (this.mesh.rotation.x > Math.PI / 2) {
    //       this.mesh.rotation.x = Math.PI / 2;
    //     }
    //     if (this.mesh.rotation.x < -Math.PI / 2) {
    //       this.mesh.rotation.x = -Math.PI / 2;
    //     }
    //     this.mouseX = e.x;
    //     this.mouseY = e.y;
    //   }
    // },

    move(e) {
      if (this.isMouseDown) {
        this.scene.rotation.y += -0.005 * (e.x - this.mouseX);
        this.scene.rotation.x += -0.005 * (e.y - this.mouseY);
        if (this.scene.rotation.x > Math.PI / 2) {
          this.scene.rotation.x = Math.PI / 2;
        }
        if (this.scene.rotation.x < -Math.PI / 2) {
          this.scene.rotation.x = -Math.PI / 2;
        }
        this.mouseX = e.x;
        this.mouseY = e.y;
      }
    },


    wheel(e) {
      if (e.wheelDelta != 0) {
        this.camera.fov += e.wheelDelta > 0 ? -1 : 1;
        if (this.camera.fov > 150) {
          this.camera.fov = 150;
        } else if (this.camera.fov < 30) {
          this.camera.fov = 30;
        }
        this.camera.updateProjectionMatrix();
      }
    },
    //全屏打开 or 关闭
    // screenButton() {
    //   this.screen = !this.screen;
    //   if (this.screen) {
    //     let element = document.documentElement;
    //     if (element.requestFullscreen) {
    //       element.requestFullscreen();
    //     } else if (element.msRequestFullscreen) {
    //       element.msRequestFullscreen();
    //     } else if (element.mozRequestFullScreen) {
    //       element.mozRequestFullScreen();
    //     } else if (element.webkitRequestFullscreen) {
    //       element.webkitRequestFullscreen();
    //     }
    //   } else {
    //     if (document.exitFullscreen) {
    //       document.exitFullscreen();
    //     } else if (document.msExitFullscreen) {
    //       document.msExitFullscreen();
    //     } else if (document.mozCancelFullScreen) {
    //       document.mozCancelFullScreen();
    //     } else if (document.webkitExitFullscreen) {
    //       document.webkitExitFullscreen();
    //     }
    //   }
    // },

    //  自适应屏幕
    selfAdaption() {
      this.renderer.setSize(window.innerWidth, window.innerHeight);// 重置渲染器输出画布canvas尺寸
      this.camera.assign = window.innerWidth / window.innerHeight; //重置相机投影的相关参数
      this.camera.updateProjectionMatrix();//渲染器执行render方法的时候会读取相机对象的投影矩阵属性projectionMatrix
      this.renderer.render(this.scene, this.camera);
    },

    //旋转打开 or 关闭
    // rotateButton() {
    //   this.panoramicRotation = !this.panoramicRotation;
    // }
  }
}

</script>

<style>

</style>