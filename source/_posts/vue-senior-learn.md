---
title: Vue文档中几个关键点的剖析
date: 2018-03-22 19:09:32
categories:
- 学习
tags:
- 学习
- Vue
---
用Vue用过一段时间之后，回顾复习Vue文档，用自己的经验加以总结，望给后来人以参考。
<!-- more -->
<!-- excerpt -->
针对Vue文档中部分大家可能不会去研读的内容，我做了个小总结，作为有经验者的快餐，不是特别适合初学者，可能有不妥之处，希望大家多提建议。

## 节省代码量的mixin

> `mixin`概念：组件级可复用逻辑，包括数据变量/生命周期钩子/公共方法，从而在混入的组件中可以直接使用，不用重复写冗余逻辑(类似继承)

使用方法: 

- 在某一公共文件夹`pub`下创建`mixin`文件夹，其下创建`mixinTest.js`

```javascript
const mixinTest = {
    created() {
        console.log(`components ${this.name} created`)
    },
    methods: {
        hello() {
            console.log('mixin method hello')
        }
    }
}
export default mixinTest
```

- 在组件中引用刚才的公共混入文件并使用

```javascript
import mixinTest from '../pub/mixin/mixinTest.js'
export default {
    data() {
        return {
            name: 'hello'
        }
    },
    mixins: [mixinTest],
    methods: {
        useMixin() {
            this.hello()
        }
    }
}

```

*ps: 若是使用`Vue.mixin()`这个方法，则会影响之后创建的所有Vue示例，慎用！*

注意`mixin`的几个特性:

1. 混入的数据变量是浅合并，冲突时以组件内的数据优先(对象里面的自定义变量)
2. 混入的生命周期函数内的逻辑会与组件内定义的生命周期函数逻辑进行合并，并且先执行(created/mounted/destroy)
3. 混入的值为对象的选项，会混合成一个对象，冲突后也是以组件内键名优先(data/method/components/directives)

## slot内容分发

> `slot`概念引入：Vue跟React在写法上的不同就在于组件与子组件内部元素的组织上，在组件里面没有`children`元素供我们访问和展现(暂不考虑`render`函数)，取而代之的API是`slot`

使用场景定义：

- 自定义的子组件里面有嵌套的HTML或者其他自定义的标签组件
- 这个自定义的子组件是写在父组件里面，嵌套的东西也放在父组件里面
- 通过在子组件的模板里面使用`<slot></slot>`标签，从而达到渲染写在父组件里的嵌套标签的效果
- 本质是把父组件放在子组件里的内容，插到了子组件的位置，多个标签也会一起被插入

```html
<template>
    <div id="app">  
        <self-component>  <!--self-component表示自定义的组件-->
            <span>12345</span>  <!--父组件里的嵌套标签--> 
        </self-component>  
    </div> 
</template>
<script>
export default {
    components: [selfComponent]
}
</script>

<!--self-component的组件模板-->
<template>
    <div>
        <button><slot></slot></button>
    </div>
</template>
<script>
export default {
    // 只有子组件的模板里面有slot标签，才能取到写在自定义组件里面的标签的渲染引用
}
</script>
```

`slot`特性的进阶两点：

1. slot插入内容的编译作用域：被分发的内容的作用域，根据其所在模板决定

- 具体内容写的位置，决定了编译的作用域(大部分情况都是在父组件作用域下)
- `2.1.0+`新增作用域插槽，从而可以把子组件的属性暴露给父组件中写在子组件内的内容使用
- 子组件中的slot标签可以直接写自定义属性，然后父组件写在slot中的标签加上`slot-scope`属性

```html
<!-- 父组件模板 -->
<child :items="items">
  <!-- 作用域插槽也可以是具名的 -->
  <li slot="item" slot-scope="props" class="my-fancy-item">{{ props.text }}</li>
</child>

<!-- 子组件模板 -->
<ul>
  <slot name="item" v-for="item in items" :text="item.text">
    <!-- 这里写当父组件引用子组件但没写内部内容时展示的东东 -->
  </slot>
</ul>
```

2. slot的`name`属性来指定标签插入的位置，也就是文档里面的具名插槽(这个[官方文档](https://cn.vuejs.org/v2/guide/components.html#%E4%BD%BF%E7%94%A8%E6%8F%92%E6%A7%BD%E5%88%86%E5%8F%91%E5%86%85%E5%AE%B9)说的明白)

- 在子组件的模板里面写的`slot`有个`name`属性(`<slot name="foo"></slot>`)
- 在父组件中写子组件里面的插槽内容，指明`slot`属性(`<p slot="foo">123</p>`)
- 父组件的内容就会对应`slot==name`放到正确的位置
- 没有指明`slot`属性的就会默认放到匿名插槽的位置上


## 动态组件

> 动态组件这个特性，很多人写的Vue项目也不少，但也没用到过这个，有必要多说几句

动态组件适用情况：

1. 单页应用，部分组件的切换不涉及路由，只是页面有一块区域的组件要变更
2. 变更的组件参数定义上是一致的，比如都是对话框，要传一个对象进去，但对象里面的数据结构不同
3. 通过使用component的is属性，避免在template中的冗余组件代码，避免多个`v-if`模板代码更加整洁

使用的方法(借鉴文档):
```html
<keep-alive>
    <component v-bind:is="currentView">
    <!-- 组件在 vm.currentview (对应组件名称)变化时改变！ -->
    <!-- 非活动组件将被缓存！可以保留它的状态或避免重新渲染 -->
    </component>
</keep-alive>
```

注意点：

- 动态切换的组件都要引入到父组件中，渲染是动态的，但引入不是。
- `<keep-alive>`包裹动态组件时，会缓存不活动的组件实例，提高性能，避免重复渲染(`keep-alive`不会渲染额外DOM结构)
- `<keep-alive>`有`include`和`exclude`这两个属性，用于指定缓存和不缓存的组件(传入字符串/数组/正则)
- 另一种避免重新渲染的方法是为标签增加属性`v-once`，用于缓存大量的静态内容，避免重复渲染。

*ps:`<keep-alive>`不会在函数式组件中正常工作，因为它们没有缓存实例。*

## 动画与过渡

> 其实很多前端工程师第一次用Vue的动画和过渡都是通过库组件来做到的，所以对这块没怎么深挖，各种过渡特效和按钮动画就跑起来了，现在就看下文档，补补课

　　前端实现动画的基本方法分为三种种：css3的过渡和keyframe/javascript操纵dom/使用webgl或者canvas来独立实现，其中第三种是作为展示动画，与交互结合较少，而Vue作为一个框架，其支持动画基是从前两种入手的，从官方文档提到的四种支持就可以看出这一点。不过官方文档是从DOM过渡和状态过渡两个方面来讲解，前者是DOM的消失和出现的动画等属性的变化，后者是页面上某些值的变化。

### DOM属性的改变

1. 若是单个元素/组件的显隐，在组件外面包裹一层`<transition></transition>`，而后选择是css过渡还是javascript过渡

- CSS过渡：
1. vue提供了六个样式后缀，本质是在dom过渡的过程中动态地添加和删除对应的`className`。(`-[enter|leave]-?[active|to]?`)
2. 如果用css库来辅助开发，可以在`transiton`这个标签上定义自定义过渡类名，也是六个属性。(`[enter|leave]-?[active|to]?-class`)
3. 常见的一种效果是元素首次渲染的动画，如懒加载图片飞入，这个时候要在`transiton`标签上加上`appear`，另有三个属性可指定(`appear-?[to|active]?-class`)

```html
<!-- 每种CSS动画库对应的class命名规则可能不同，所以根据不同库要自己写，以animate.css为例 -->
<transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
    :duration="{ enter: 500, leave: 800 }"
>...</transition>
<!-- duration属性可以传一个对象，定制进入和移出的持续时间-->
```

- JS过渡：
1. 因为现在很多动画库需要工程师调用库提供的函数，把dom元素传入进行处理，这个时候需要这种方式
2. 通过在`transiton`这个标签上添加监听事件，共8个(`[before|after]?-?[enter|leave]-?[cancelled]?`)
3. 监听事件的回调函数的第一个参数都是el，为过渡的dom元素，在`enter`和`leave`这两个还会传入`done`作为第二个参数
4. 元素首次渲染的动画，可以指定的监听事件有4个(`[before|after]?-?appear`和`appear-cancelled`)

```html
<template>
    <transition v-bind:css="false"
    v-on:before-enter="beforeEnter" v-on:enter="enter"
    v-on:leave="leave" v-on:leave-cancelled="leaveCancelled">
        <!-- 对于仅使用 JavaScript 过渡的元素添加 v-bind:css="false"，Vue 会跳过 CSS 的检测 -->
    </transition>
</template>
<script>
methods: { // 以Velocity库为例
    beforeEnter: function (el) {/*...*/},
  // 此回调函数是可选项的设置
  enter: function (el, done) {
    // Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
    done()  //回调函数 done 是必须的。否则，它们会被同步调用。
  },
  leave: function (el, done) {
    // Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
    done()
  },
  leaveCancelled: function (el) {/*...*/}
}
</script>
```

*多元素过渡其实就是一句话：照常使用`v-if/v-else`的同时对同一种标签加上`key`来标识*

Vue对于这种多元素动画有队列上的处理，这就是`transiton`这个标签上的`mode`属性，通过指定(in-out|out-in)模式，实现消失和出现动画的队列效果，让动画更加自然。

```html
<transition name="fade" mode="out-in">
  <!-- ... the buttons ... -->
</transition>
```

*多组件过渡也是一句话：用上一节提到的动态组件，即可完成。*

针对列表过渡，其本质仍是多个元素的同时过渡，不过列表大部分是通过数组动态渲染的，因此有独特的地方，不过整体的动画思路不变。具体有以下几点

1. 使用`transitoin-group`这个组件，其需要渲染为一个真实元素，可以通过tag这个属性来指定。
2. 列表的每个元素需要提供`key`属性
3. 使用`CSS`过渡的话，要考虑到列表内容变化过程中，存在相关元素的定位改变，如果要让定位是平滑过渡的动画，要另外一个`v-move`属性。
这个属性是通过设置一个`css`类的样式，来将创建元素在定位变化时的过渡，Vue内部是通过`FLIP`实现了一个动画队列，只要注意一点就是过渡元素不能设置为`display:inline`,这里需要文档上的代码做一个简短的`demo`：(其实通过在li上设置过渡transition属性也可以实现`v-move`的效果)

```html
<template>
    <button v-on:click="shuffle">Shuffle</button>
    <transition-group name="flip-list" tag="ul">
        <li v-for="item in items" v-bind:key="item">{{ item }}</li>
    </transition-group>
</template>
<script>
import _ from 'lodash';
export default {
    data() {
        return {
            items: [1,2,3,4,5,6,7,8,9]
        }
    },
    methods: {
        shuffle: function () {
            this.items = _.shuffle(this.items)
        }
    }
}
</script>
<style lang="css">
.flip-list-move {
  transition: transform 1s;
}
</style>
```

### 数值和属性动态变化

这一部分的动画主要是针对数据元素本身的特效，比如数字的增减，颜色的过渡过程控制，svg动画的实现等，其本质都是数字/文本的变化。
我自己总结就是：通过利用Vue的响应式系统，把数字的变化通过外部库把DOM上对应数值的变化做出连续的效果，如1->100是个数字递增的连续过程，黑色->红色的过程。官方文档主要是用几个示例代码来说明，其本质步骤如下：

1. 在页面上通过`input`的双向绑定修改某一变量a，还有一个处理dom上的过渡效果的变量b
2. 这个数据被watcher绑定(watch对象中某个属性是这个变量a)，触发逻辑
3. 在watcher里面的逻辑就是通过外部过渡库，指定初始值b和最终值a，是把b的值最后改为a
4. DOM上绑定的变量就是b，如果某些复杂情况可能是基于b的计算属性，从而把b的变化过程展现出来

　　上面这个思路走一遍下来就完成了一个单元级别的动画效果，这种类似的流程其实是很常见的需求，所以有必要把这个过程封装成一个组件，只暴露要过渡的值作为入口，每次改变这个值都是一个动画过渡效果。组件封装需要在上面四个步骤的基础上添加`mounted`生命周期规定初始值即可，同时原来的两个值a/b在组件里面作为一个值，可以用`watch`对象中的`newValue`和`oldValue`作为区分。
　　至于最后的SVG，其本质也是数字的过渡，只不过里面涉及的状态变量更多，代码更长而已，不过纯前端页面这种需求倒还是不多的，不过作为爱好倒可以鼓捣一些好玩的小`demo`，不过肯定需要设计师的参与，要不那些参数可不好调出来呐。