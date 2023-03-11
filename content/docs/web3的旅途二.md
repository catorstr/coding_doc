# web3前端篇three.js 的学习

```js
#接上一节main.js文件
import * as THREE from "three"
//console.log(THREE)

//three.js的最基本的内容

//1.创建场景
const scene = new THREE.Scene();
//2.创建相机
//相机角度75 宽高比 近端0.1 远端100
const camera = new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000);//创建透视相机
//设置相机对象的位置
camera.position.set(0,0,10);
//将相机添加到场景中
scene.add(camera);
//3.创建物体
//创建一个长宽高位1的立方体
const cube = new THREE.BoxGeometry(1,1,1);
//设置物体的材质
const texture = new THREE.MeshBasicMaterial({color:0xffccff});
const cube2 = new THREE.Mesh(cube,texture)
//将立方体添加到场景中
scene.add(cube2);

//初始化渲染器
const renderer = new THREE.WebGLRenderer();
//将渲器进行渲染
//设置渲染的尺寸大小
renderer.setSize(window.innerWidth,window.innerHeight);
//将webGl渲染的canvas(画布)内容添加到body中
//console.log(renderer)
document.body.appendChild(renderer.domElement);
//使用渲染器 通过相机将场景渲染进来
renderer.render(scene,camera);

```
## 使用控制器去查看3d物体

```js


```