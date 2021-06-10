## DOM

**HTML DOM**

```
HTML的标准对象模型
HTML的标准编程接口
HTML DOM定义了所有HTML元素的对象和属性，以及访问它们的方法
```

**XML DOM**

```
XML DOM定义了所有元素的对象和属性，以及访问它们的方法
```

------

# Vue.js

## Vue实例

### 构造器

每个Vue.js应用都是通过**构造函数Vue()**创建出来一个Vue的**根实例**,以及可选的嵌套的、可复用的**组件**树组成的

```vue
var vm = new Vue({
	//选项
})
```

### 数据和方法

当一个Vue实例被创建时，它将data对象中的所有的property加入到Vue的响应式系统中，当property的值发生改变时，视图将会产生响应

```
Object.freeze(property)会阻止修改现有的property值，意味值响应系统无法追踪变化
```

### 实例周期钩子

生命周期钩子函数会在不同阶段执行

```
new Vue({
  data:{
    a: 1
  },
  created:function(){
    console.log('a is' + this.a)
  }
})

mounted、updated、destroyed
生命周期钩子的this上下文指向调用它的Vue实例
```

## 模版语法

### 文本

```
数据绑定使用"Mustache"语法（双打括号）的文本插值
<span>Message:{{msg}}</span>
Mustache标签将会被替代为对应数据对象（data)上的msg属性的值
v-once指令能使插值处的内容不会更新
双打括号会将数据解释为普通文本，而非HTML代码
```

### 原始HTML

```
输出真正的HTML，需要使用v-html指令
```

### Attribute

```
Mustache语法不能作用在HTML attribute上，可使用v-bind指令为HTML元素的属性赋值（若属性值为null、undefined、false,元素不会被渲染）
```

### 使用JavaScript表达式

```
{{msg+1}}、{{ok ? 'yes':'no'}}
```

## 指令

指令是带有`v-`前缀的特殊attribure,指令的职责是，当表达式的值改变时，将其产生的影响，响应式作用于DOM

### 参数

```
一些指令能够接受一个参数，在指令名称之后以冒号表示，用于响应式更新HTML元素的属性
<a v-bind:hred="url"></a>
<a v-on:click="check"></a>
```

## 计算属性

```
var vm = new Vue({
  computed: {
    reversedMessage: function(){
    //this指向vm实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

**v-bind：动态绑定属性** 语法糖(:)

**v-on:绑定事件监听器** 语法糖(@)

