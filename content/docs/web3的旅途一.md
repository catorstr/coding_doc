# web3前端篇
## vite脚手架的安装
```go
yarn create vite
```
>此命令会帮助我们全局安装一个东西：create-vite(vite的脚手架)
create-vite内置了vite，我们使用vite 来构建我们的工程。

接下来正式构建我们的工程项目
```bash
mkdir demo
cd demo
yarn init -y #(或者 npm init -y) 初始化
yarn add vite -D #将vite引入我们的项目当中
```
执行完后会生成一个package.json,我们在这个文件中添加:
```json
  "scripts": {
    "dev": "vite", // 启动开发服务器，别名：`vite dev`，`vite serve`
    "build": "vite build", // 为生产环境构建产物
    "preview": "vite preview" // 本地预览生产构建产物
  }
```
来配置开发命令，就可以直接使用yarn dev 或npm run dev来启动我们的项目了。
不在启动项目之前，我们需要进行一些项目的配置和页面书写。
## vite的配置
```js
#根目录创建vite.config.js文件
optimizeDeps:{
    exclude:[], //将数组中指定的依赖不进行依赖构建
}
```
> 1.配置vite配置文件的语法提示:
```js
#在vite.config.js文件中
import {defineConfig} from "vite";
import viteBaseConfig from "./vite.base.config.js"
import viteDevConfig from "./vite.dev.config.js"
import viteProdConfig from "./vite.prod.config.js"
const envResolver ={
    "build":() =>{
        console.log("生产环境...")
        return Object.assign({},viteBaseConfig,viteProdConfig)
    },
    "serve":() => {
        console.log("开发环境...")
        return  Object.assign({},viteBaseConfig,viteDevConfig)
    }
}
export default defineConfig( ({command})=>{ //箭头函数，在函数中转入一个对象
    console.log("command",command);
    return envResolver[command]();
    // //(parameter) command: "build" | "serve"
    // if (command ==="build"){
    //     //代表生产环境的配置vite.base.config.js 书写方式和这个当前文件的差不多
    //     //vite.prod.config.js
    // }else{
    //     //代表开发环境的配置 vite.dev.config.js
    // }
    
})
```
## CSS 预处理器(选一个即可)
```bash
# .scss and .sass
yarn add -D sass 
# .less
npm add -D less
# .styl and .stylus
npm add -D stylus
```
## 开始构建我们的工程目录
```bash
touch index.html
```
```html
# index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="./src/assets/css/style.css">
</head>
<body>
    <h1>hello</h1>
    <!-- 导入入口文件 并设置模块化开发 -->
    <script src="./main.js" type="module"></script>
</body>
</html>

```

```css
//mkdir src/assets/css/style.css
*{
    margin: 0;
    padding: 0;
}
body{
    border-color: skyblue;
}
```
```js
#touch vite.base.config.js
import {defineConfig} from "vite";
export default defineConfig({
    optimizeDeps:{
        exclude:[],
    }
})
```
```js
#touch vite.dev.config.js
import {defineConfig} from "vite";
export default defineConfig({})
```
```js
touch vite.prod.config.js
import {defineConfig} from "vite";
export default defineConfig({})
```
## 引人three.js
```bash
yarn add gsap dat.gui three
mkdir src/assets/imgs
```
```js
# touch main.js
import * as THREE from "three"
console.log(THREE) //查看three.js是否引入成功

```