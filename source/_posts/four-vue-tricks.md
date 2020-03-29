---
title: 四个Vue的写法优化技巧
date: 2018-04-11 12:45:11
categories:
- 工作
- 学习
tags:
- vue
- webpack
---
最近看到了`Youtobe`上的视频，某大神总结了几个`Vuejs`的优化方法，取名为`Vue`文档不想让你知道的技巧，这个标题比较浮夸，我对其内容进行了提炼，总结了四点，希望对大家有帮助。

<!-- more -->
<!-- excerpt -->

## watch的优化写法

> 我们平时总会遇到组件创建时获取列表，筛选项改变后刷新列表的需求，在`watch`中的逻辑，还要在组件的`created()`中再执行一遍，以下写法解决此痛点。

*`bad`*

```javascript
export default {
  created() {
    this.fetchListData()
  },
  watch: {
    searchValue() {
      this.fetchListData()
    }
  }
}
```

*`good`*

```javascript
export default {
  watch: {
    searchValue: {
      handler: 'fetchListData',
      immediate: true  //表示组件初始化时执行一次
    }
  }
}
```

## 优化组件引入的代码

> 在开发大型应用时，很多页面用到很多公共UI库的组件，按照Vue原本的写法，每种组件都要在每个页面引用一次，真是好累，如何省事偷懒呢？

*`bad`*

```javascript
import BaseButton from '../components/baseButton'
import BaseInput from '../components/baseInput'
import BaseCard from '../components/baseCard'

export default {
  components: {
    BaseButton,
    BaseInpur,
    BaseCard
  }
}
```

*`good`*

在公共组件`components`文件夹下新建一个`allImport.js`文件，本质是使用webpack的一个`require.context()`方法来实现代码层级的动态引入，下面注意代码的实现，重点看注释。

```javascript
// allImport.js
import Vue from 'vue'

function capitalizeFirstLetter(string) {
  // 此函数用途是首字符大写，baseInput->BaseInput 如果是已经首字符大写的文件名，不需要
  return string.charAt(0).toUpperCase() + string.slice(1)
}

function handleFileName(string) {
  // 把形如 ./fileName.vue -> fileName
  return string.replace(/^\.\//, '').replace(/\.\w+&/, '')
}

const requireComponent = require.context('.', false, /\.vue$/)
// 表示找到此文件夹下所有.vue为后缀的文件，详见webpack require.context文档

requireComponent.keys().forEach(fileName => { // fileName可以认为是路径字符串
  const componentConfig = requireComponent(fileName)
  const componentName = capitalizeFirstLetter(handleFileName(fileName))
  Vue.component(componentName, componentConfig.default || componentConfig)
})
```

在`main.js`中引入这个脚本即可 `import 'components/global.js'`
PS: 如果是用`extend`这个方法来继承Vue实例实现节省代码，每个页面的`import`还是不能省略

## 父子组件通信的优化写法

> 我们Vue组件写多了就会感觉到，父组件上面要写上监听子组件emit来的事件，同时每一个属性都得绑定了传过去，无论自定义的还是click/placeholder之类原生的。这个优化方法是把一些原生得属性和事件进行简写，尤其对组件库之类改造原有HTML元素有奇效。

*`bad`*

```HTML
<!-- 父组件的模板写法 -->
<template>
  <div>
  <!-- 可以看到这个自定义的组件上有好多原生的属性和事件 -->
    <BaseInput :value="value"
    label="密码" placeholder="place input your password"
    @input="handleInput" @focus="handleFocus">
    </BaseInput>
  </div>
</template>

<!-- 子组件写法 -->
<template>
  <label>
    {{lable}}
    <input :value="value"
      :placeholder="placeholder"
      @focus="$emit('focus', $event)"
      @input="$emit('input', $event.target.value)"
    >
    <!-- 这些原生事件实质是被代理的一层，从而导致了很多的冗余代码，其实我们可以利用两个简写手段，避免这种无意义的代理，直接传入 -->
  </label>
</template>
```

- 优化点1：Dom的原生属性可以通过`$attrs`这个`API`来实现非声明式传递。`$attr`包含的是父作用域中不作为`prop`被识别的(`class`/`style`除外)特性。
- 优化点2：Dom的原生事件可以通过`$listeners`实现非声明式传入，`$listeners`包含了父作用域中不含`.native`修饰器的事件监听(v-on:),如果是更多层级，可以通过`v-on="$listeners"`一层层传递到组件内部。
- 注意：默认父作用与不被认为是`props`的属性将会回退到子组件的根元素上，也就是例子中的`label`上，需在当前子组件的Vue内部设置`inheritAttrs: false`

*`good`*

```html
<!-- 父组件保持不变，子组件优化写法如下 -->
<template>
    <input :value="value" v-bind="$attrs" v-on="selfListeners">
</template>
<script>
  export default {
    inheritAttrs: false,
    computed: {
      selfListeners() {
        return {
          ...this.$listeners,
          input: event => this.$emit('input', event.target.value)
        }
      }
    }
  }
</script>
```

## 使用Render函数来组织渲染逻辑

> 教程中的例子是关于省略掉根元素来考虑的，其实这种写法应该是更贴近`React`的思路，所以学会用`js`来生成`HTML`吧

*函数式组件的`render`的写法*

```javascript
// 注意，如果使用这种JSX的写法，要引入`babel-plugin-transform-vue-jsx`这个插件
// 根据最新的这个插件文档，已经不用在第一个参数上传递`createElement`这个h参数了
Vue.component('my-router', {
  functional: true,
  // context参数代表上下文，里面有属性props
  render(context) {
    return context.props.routes.map(route =>
      <li key={route.name}>
        <router-link to={route}>{route.title}</router-link>
      </li>
    )
  }
})
```

<b>如果这篇文章对您有帮助，欢迎微信打赏~。</b>

打赏二维码：
![](four-vue-tricks/dashang.jpg)

打赏获取特权流程：
- 扫描下发二维码 加我为好友
- 发送打赏截图给我
- 拉你进我的微信学习群
- 解答你工作中遇到的Vue问题

![](four-vue-tricks/wechat.jpg)