# VR全景看房
## 引言
### 系统背景
&emsp;&emsp;本人因个人兴趣以 *three.js* 编写的 *VR全景看房DEMO*。
### 系统目的
&emsp;&emsp;以备不时之需。
## 项目简介
### 项目全称
&emsp;&emsp;VR全景看房
### 软件架构
* Vue 2.6.x、three 0.130.1、Element 2.15.3（详见：package.json）
## 命令
### 依赖引入
```
npm install
```
### 开发模式
```
npm run dev
```
### 生产编译
```
npm run build
```
## 目录说明
* vr-panorama：vr-panorama总目录
  * public：公开域
    * static：公共静态文件
      * music：背景音乐
      * room：房间全景照片
  * src：源码
    * components：组件库
      * panorama.vue：全景组件
    * lib：依赖css or js
      * css：css依赖
    * static：静态资源
      * button：按钮图标
    * App.vue：组件App调用页面
    * main.js：入口文件
  * .gitignore：忽略提交配置
  * package.json：配置文件
  * package-lock.json：package-lock.json
  * README.md：自述文件