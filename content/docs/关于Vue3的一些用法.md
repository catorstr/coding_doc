# 关于Vue3的一些用法

## 基于setup语法糖

> **直接在script标签中添加setup属性就可以直接使用setup语法糖了。**
> 使用setup语法糖后，**不用写setup函数；组件只需要引入不需要注册；属性和方法也不需要再返回，可以直接在template模板中使用**。
>
> ```vue
> <template>
> 		<my-component @click="func" :numb="numb"></my-component>
> </template>
> <script lang="ts" setup>
> import {ref} from 'vue';
> import myComponent from '@/component/myComponent.vue';
> //此时注册的变量或方法可以直接在template中使用而不需要导出
>     const numb = ref(0);
>     let func = ()=>{
>         numb.value++;
>     }
> </script>
> 
> ```
>
> 

### 父组件向子组件传值

> **defineProps** 在默认的语法中父组件向子组件传值的时候，子组件是通过props来接收的，在setup语法糖中需要使用defineProps 。

1. 父组件如何传值

```vue
<template>
	<my-component @click="func" :numb="numb"></my-component>
</template>
<script lang="ts" setup>
import {ref} from 'vue';
import myComponent from '@/components/myComponent.vue';
		const numb = ref(0);
		let func = ()=>{
			numb.value++;
		}
</script>

```

2.  子组件接收和使用

```vue
<template>
	<div>{{numb}}</div>
</template>
<script lang="ts" setup>
    import {defineProps} from 'vue';
    defineProps({
        numb:{
            type:Number,
            default:NaN
        }
    })
</script>

```

### 子组件向父组件传值

> vue3的setup语法糖使用的是defineEmits。vue3的子传父方式如下所示： 

1.  子组件的传递方式

```vue
<template>
  <button @click="clickChild">点击子组件</button>
</template>
 
<script setup>
import { defineEmits } from 'vue'
// 使用defineEmits创建名称，接受一个数组
const emit = defineEmits(['clickChild'])
const clickChild=()=>{
  let param={
    content:'b'
  }
  //传递给父组件
  emit('clickChild',param)
}
</script>
 
<style>
 
</style>
```

2. 父组件接收与使用

```vue
<template>
  <div class="hello">
  我是父组件
  <!-- clickChild是子组件绑定的事件，click是父组件接受方式 -->
   <Child  @clickChild="clickEven"></Child>
 <p>子组件传递的值是 {{result}}</p>
 </div>
</template>
 
<script setup>
import Child from './Child'
import {ref} from 'vue'
const result=ref('')
const clickEven=(val)=>{
  console.log(val);
  result.value=val.content
}
</script>
 
<style scoped>
 
</style>
```

### 子组件调用父组件中的方法

1. 子组件代码

```vue
<template>
	<div>{{numb}}</div>
	<button @click="onClickButton">数值加1</button>
</template>
<script lang="ts" setup>
    import {defineProps,defineEmits} from 'vue';
    defineProps({
        numb:{
            type:Number,
            default:NaN
        }
    })
    const emit = defineEmits(['addNumb']);
    const onClickButton = ()=>{
        //emit(父组件中的自定义方法,参数一,参数二,...)
        emit("addNumb");
    }
</script>
```

2. 父组件代码

```vue
<template>
	<my-component @addNumb="func" :numb="numb"></my-component>
</template>
<script lang="ts" setup>
		import {ref} from 'vue';
		import myComponent from '@/components/myComponent.vue';
		const numb = ref(0);
		let func = ()=>{
			numb.value++;
		}
</script>
```

### 父组件获取子组件中的属性值

>  defineExpose：子组件暴露属性，可以在父组件中拿到

1. 子组件的传递方式

```vue
<template>
	<div>子组件中的值{{numb}}</div>
	<button @click="onClickButton">数值加1</button>
</template>
<script lang="ts" setup>
import {ref,defineExpose} from 'vue';
    let numb = ref(0);
    function onClickButton(){
        numb.value++;	
    }
    //暴露出子组件中的属性
    defineExpose({
        numb 
    })
</script>
```

2. 父组件显示方式

```vue
<template>
	<my-comp ref="myComponent"></my-comp>
	<button @click="onClickButton">获取子组件中暴露的值</button>
</template>
<script lang="ts" setup>
import {ref,onMounted} from 'vue';
import myComp from '@/components/myComponent.vue';
    //注册ref，获取组件
    const myComponent = ref();
    function onClickButton(){
        //在组件的value属性中获取暴露的值
        console.log(myComponent.value.numb)  //0
    }
    //注意：在生命周期中使用或事件中使用都可以获取到值，
    //但在setup中立即使用为undefined
    console.log(myComponent.value.numb)  //undefined
    const init = ()=>{
        console.log(myComponent.value.numb)  //undefined
    }
    init()
    onMounted(()=>{
        console.log(myComponent.value.numb)  //0
    })
</script>
```

