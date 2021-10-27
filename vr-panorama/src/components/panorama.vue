<template>
  <div class="app-combination" v-loading="loading">
    <div id="3d" ref="3d"></div>
    <div id="choice" v-if="!loading">
      <div class="tips">
        <div style="margin-top: 17px; width: 60px; text-align: center;">{{ this.index }}/{{ this.total }}</div>
      </div>
      <div style="position: absolute; right: 20px; top: 20px;">
        <button type="button" class="el-button el-button--danger is-circle" @click="soundButton()">
          <span v-if="sound">
            <i>
              <img src="../static/button/sound/openSound.png" alt="" height="20" width="20">
            </i>
          </span>
          <span v-else>
            <i>
              <img src="../static/button/sound/turnSoundOff.png" alt="" height="20" width="20">
            </i>
          </span>
        </button>
        <button type="button" class="el-button el-button--danger is-circle" @click="screenButton()">
          <span v-if="!screen">
            <i>
              <img src="../static/button/screen/fullScreen.png" alt="" width="18" height="18">
            </i>
          </span>
          <span v-else>
            <i>
              <img src="../static/button/screen/outScreen.png" alt="" width="18" height="18">
            </i>
          </span>
        </button>
        <button type="button" class="el-button el-button--danger is-circle" @click="rotateButton()">
          <span v-if="panoramicRotation">
            <i>
              <img src="../static/button/rotate/rotate.png" alt="" width="20" height="20">
            </i>
          </span>
          <span v-else>
            <i>
              <img src="../static/button/rotate/noRotate.png" alt="" width="20" height="20">
            </i>
          </span>
        </button>
      </div>
      <div class="carousel">
        <button
            type="button"
            style="position: absolute;left: 5px"
            class="el-button el-button--danger is-circle"
            @click="onClickCarousel('left')"
        >
          <i class="el-icon-arrow-left"></i>
        </button>
        <button
            type="button"
            style="position: absolute;right: 5px;"
            class="el-button el-button--danger is-circle"
            @click="onClickCarousel('right')"
        >
          <i class="el-icon-arrow-right"></i>
        </button>
      </div>
      <div class="menu">
        <el-button
            type="primary"
            size="small"
            plain
            :class="{active:model===1}"
            @click="onClickButton('theRestaurant',1,6)"
        >
          客餐厅
        </el-button>
        <el-button
            type="primary"
            size="small"
            plain
            :class="{active:model===2}"
            @click="onClickButton('saloon',2,4)"
        >
          客厅
        </el-button>
        <el-button
            type="primary"
            size="small"
            plain
            :class="{active:model===3}"
            @click="onClickButton('kitchen',3,3)"
        >
          厨房
        </el-button>
        <el-button
            type="primary"
            size="small"
            plain
            :class="{active:model===4}"
            @click="onClickButton('bedroom',4,4)"
        >
          卧室
        </el-button>
        <el-button
            type="primary"
            size="small"
            plain
            :class="{active:model===5}"
            @click="onClickButton('bathroom',5,2)"
        >
          浴室
        </el-button>
        <el-button
            type="primary"
            size="small"
            plain
            :class="{active:model===6}"
            @click="onClickButton('study',6,3)"
        >
          书房
        </el-button>
      </div>
    </div>
  </div>
</template>
<style>
@import "../lib/css/panorama.css";
</style>
<script>
import * as THREE from 'three'
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'

export default {
  name: "panorama",
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
    };
  },
  //构造函数
  created() {
    this.$nextTick(() => {
      this.defaultRun();
    })
  },
  mounted() {
    //全景自适应屏幕大小
    let _this = this;
    window.onresize = function () {
      _this.selfAdaption();
    };
    this.initAudio();
  },
  //普通方法
  methods: {
    //全景默认启动
    defaultRun() {
      if (!this.first) {
        this.initScene();
        this.initCamera();
        this.initCamera();
        this.initPanorama();
        this.initRenderer();
        this.refresh();
        this.initControls();

      }
    },
    //场景生成
    initScene() {
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color("#fff");
    },
    //相机生成
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.001, 10000);
      this.camera.zoom = 1;
      this.camera.updateProjectionMatrix();
      this.camera.position.set(-0.87, 0.03, 0.4);
      this.camera.lookAt(this.scene.position);
    },
    //生成全景
    initPanorama() {
      setTimeout(() => {
        let textureLoader = new THREE.TextureLoader().load('./static/room/theRestaurant/' + this.index + '.jpg', () => {
          this.loading = false;
          let box = new THREE.SphereGeometry(100, 100, 100);//创建全景球体
          let material = new THREE.MeshBasicMaterial({color: 0xffffff, side: THREE.BackSide,}); //全景球体样式
          this.mesh = new THREE.Mesh(box, material);
          this.mesh.material.map = textureLoader;
          this.scene.add(this.mesh);
        });
      }, 800);
    },
    //生成渲染
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({antialias: true,});
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.$refs["3d"].appendChild(this.renderer.domElement);
    },
    //生成控件对象
    initControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);//创建控件对象
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
      requestAnimationFrame(this.refresh);
    },
    //背景音乐
    initAudio() {
      let _this = this;
      let audioLoader = new THREE.AudioLoader();
      _this.audio = new THREE.Audio(new THREE.AudioListener());
      audioLoader.load('../static/music/backgroundMusic.mp3', function (audioBuffer) {
        _this.audio.setBuffer(audioBuffer);
        _this.audio.setLoop(true);
        _this.audio.setVolume(1);
        //_this.audio.play();
      });
    },
    //自适应屏幕
    selfAdaption() {
      this.renderer.setSize(window.innerWidth, window.innerHeight);// 重置渲染器输出画布canvas尺寸
      this.camera.assign = window.innerWidth / window.innerHeight; //重置相机投影的相关参数
      this.camera.updateProjectionMatrix();//渲染器执行render方法的时候会读取相机对象的投影矩阵属性projectionMatrix
      this.renderer.render(this.scene, this.camera);
    },
    //左右切换场景
    onClickCarousel(tag) {
      this.index = ('left' === tag) ? (((this.index - 1) < this.small) ? this.total : this.index - 1) : (((this.index + 1) > this.total) ? this.small : this.index + 1);
      let textureLoader = new THREE.TextureLoader().load('./static/room/' + this.modelName + '/' + this.index + '.jpg', () => {
        this.mesh.material.map = textureLoader;
        this.camera.updateProjectionMatrix();
      });
    },
    //场景切换按钮
    onClickButton(path, model, modeTotal) {
      if (model !== this.model) {
        this.model = model;
        this.total = modeTotal;
        this.index = 1;
        this.modelName = path;
        let textureLoader = new THREE.TextureLoader().load('./static/room/' + path + '/1.jpg', () => {
          this.mesh.material.map = textureLoader;
          this.camera.updateProjectionMatrix();
        });
      }
    },
    //声音打开 or 关闭
    soundButton() {
      this.sound = !this.sound;
      if (this.sound) {
        this.audio.play();
      } else {
        this.audio.stop();
      }
    },
    //全屏打开 or 关闭
    screenButton() {
      this.screen = !this.screen;
      if (this.screen) {
        let element = document.documentElement;
        if (element.requestFullscreen) {
          element.requestFullscreen();
        } else if (element.msRequestFullscreen) {
          element.msRequestFullscreen();
        } else if (element.mozRequestFullScreen) {
          element.mozRequestFullScreen();
        } else if (element.webkitRequestFullscreen) {
          element.webkitRequestFullscreen();
        }
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        } else if (document.msExitFullscreen) {
          document.msExitFullscreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.webkitExitFullscreen) {
          document.webkitExitFullscreen();
        }
      }
    },
    //旋转打开 or 关闭
    rotateButton() {
      this.panoramicRotation = !this.panoramicRotation;
    }
  }
};
</script>

