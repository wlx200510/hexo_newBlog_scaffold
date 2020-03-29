---
title: Vue组件库开发技巧
date: 2020-03-29 16:28:57
categories:
- 技术总结
tags:
- JS
- VUE
---
讲几个开发`vue`组件中用的比较少，但恰当使用可以提升可读性和维护性的技巧。以下讲解都带有例子
<!-- more -->

## 函数式组件——包装组件

函数式组件，没有管理任何状态，也没有监听任何传递给它的状态，也没有生命周期方法。实际上，它只是一个接受一些 prop 的函数。在这样的场景下，我们可以将组件标记为 `functional`，这意味它无状态 (没有[响应式数据](https://cn.vuejs.org/v2/api/#选项-数据))，也没有实例 (没有 `this` 上下文)。

组件需要的一切都是通过 `context` 参数传递，它是一个包括如下字段的对象：

- `props`：提供所有 prop 的对象
- `children`: VNode 子节点的数组
- `slots`: 一个函数，返回了包含所有插槽的对象
- `scopedSlots`: (2.6.0+) 一个暴露传入的作用域插槽的对象。也以函数形式暴露普通插槽。
- `data`：传递给组件的整个[数据对象](https://cn.vuejs.org/v2/guide/render-function.html#深入数据对象)，作为 `createElement` 的第二个参数传入组件
- `parent`：对父组件的引用
- `listeners`: (2.3.0+) 一个包含了所有父组件为当前组件注册的事件监听器的对象。这是 `data.on` 的一个别名。
- `injections`: (2.3.0+) 如果使用了 [`inject`](https://cn.vuejs.org/v2/api/#provide-inject) 选项，则该对象包含了应当被注入的属性。

在添加 `functional: true` 之后，需要更新我们的锚点标题组件的渲染函数，为其增加 `context` 参数，并将 `this.$slots.default` 更新为 `context.children`，然后将 `this.level` 更新为 `context.props.level`。

### 函数式组件的特点

函数式组件没有生命周期、无状态、没有实例，那就意味着在性能上它渲染速度更快，开销更低。在业务代码上可以清楚地表明当前组件的轻逻辑性，便于复用。在组件库中作为包装组件，动态渲染(类`component`组件)。

下面的示例是用一个包装组件来表示函数式组件灵活的用法，通过属性实现动态组件。

```javascript
export default { 
  functional: true, 
  render: function (createElement, context) { 
    return createElement(context.props.domType, context.data, context.children); 
  } 
}
```

```vue
<template>
  <div>
    <functionnalScript domType="model" style="width:100%"></functionnalScript>
  </div>
</template>

<script>
import customizeTable from "../components/customizeTable.vue";
import model from "../components/model.vue";
import functionnalScript from "../components/functionnal.js";
export default {
  name: "pageA",
  components: { 
      customizeTable, 
      model, 
      functionnalScript: functionnalScript 
  }
};
</script>
```

正常调用的时候父页面我们有customizeTable组件和model组件我们可以通过在一个属性`domType`指定functionnalScript显示对应的组件，而且可以在多处复用。

```vue
// 模板的函数式组件 functional.vue
<template functional>
  <div>
    <button v-bind="data.attrs" v-on="listeners">
      <slot />
    </button>
  </div>
</template>
```

模板的函数式组件functional组件就实现了完全透传任何 `attribute`、`事件监听器`、`子节点`等

##  组件间的属性透传——解构运算符

当项目中引入某些自定义参数非常多的组件，并且在业务中需要对这种基础组件进行二次封装时，使用属性透传可以让整个组件的扩展非常的优雅和可控。

```javascript
<script>
import { Table } from "iview";
export default {
  name: "customizeTable",
  props: { ...Table.props, test: { type: [String], default: "扩展属性" } },
  data() {
    return { defaultProps: { border: true, stripe: true } };
  },
  render(h) {
    var props = Object.assign({ ...this._props }, this.defaultProps);
    return h(Table, { props: props });
  }
};
</script>
```

不使用函数式组件的写法也可以实现，下面的写法我个人更加常用：

```javascript
<template>
  <div>
    <Table v-bind="{...mergeProps}"></Table>
  </div>
</template>
<script>
import { Table } from "iview";
export default {
  name: "customizeTable",
  props: { 
      ...Table.props,
      test: { 
          type: [String],
          default: "扩展属性" 
      } 
  },
  computed: {
    mergeProps: function() {
      return Object.assign({ ...this._props }, this.defaultProps);
    }
  },
  data() {
    return { 
        defaultProps: { 
            border: true, 
            stripe: true 
        } 
    };
  }
};
</script>
```

首先我们通过`Table`组件的props属性获取的Table所有默认参数，然后通过`this._props`实现属性透传，同时我们还可以在`props`中扩展你需要的属性如test。函数式写法中中绑定属性我们使用了`render`方法，第二种写法我们通过`v-bind`实现绑定对象中所有的属性。

## 递归组件——加载入口

递归组件在我们日常开发中的发挥空间还是很大的，比如我们的菜单，评论等有层级的功能都可以使用到它。我们来看一个菜单的例子：

```vue
<template>
  <div>
    <div v-for="item in menuList">
      <div class="hasChildren">{{item.name}}
        <Icon v-if="item.children" type="ios-arrow-down" />
      </div>
      <side-menu v-if="item.children" :menuList="item.children"></side-menu>
    </div>
  </div>
</template>

<script>
export default {
  name: "SideMenu",
  props: {
    menuList: {
      type: Array,
      default() {
        return [];
      }
    }
  }
};
</script>

<style>
.hasChildren {
  background: #999;
  color: #fff;
  height: 40px;
  line-height: 40px;
}
</style>
```

以下是调用`sideMenu`组件的代码：

```vue
<template>
  <div>
    <sideMenu :menuList="menuList"></sideMenu>
  </div>
</template>
<script>
import sideMenu from "../components/side-menu.vue";
export default {
  name: "pageA",
  components: { sideMenu },
  data() {
    return {
      menuList: [
        { name: "side1", children: [{ name: "side1-1" }, { name: "side1-2" }] },
        {
          name: "side2",
          children: [
            { name: "side2-1" },
            { name: "side2-2", children: [{ name: "side2-2-1" }] }
          ]
        }
      ]
    };
  }
};
</script>
```

我们可以看到在`sideMenu`组件内部调用了它本身，这就是递归组件的基本使用方式。在更加复杂一点的情况就是有2个组件A和B，他们之间互有调用关系。这时会出现一个问题，到底是先有鸡还是先有蛋。

模块系统发现它需要 A，但是首先 A 依赖 B，但是 B 又依赖 A，但是 A 又依赖 B，如此往复。这变成了一个循环，不知道如何不经过其中一个组件而完全解析出另一个组件，解决方法有2个。

```javascript
//方法一 在A组件
beforeCreate: function () {
  this.$options.components.sideMenu = require('./sideMenu.vue').default
}
//方法二 异步加载sideMenu组件
components: {
  sideMenu: () => import('./sideMenu.vue')
}
```



## 全局函数调用组件

把组件的创建和销毁封装到函数方法中，可以参考`vue-create-api`的源码来进行处理。**一个能够让 Vue 组件通过 API 方式调用的插件**。( vue-create-api [源码地址](https://github.com/cube-ui/vue-create-api) )

```javascript
import CreateAPI from 'vue-create-api'

Vue.use(CreateAPI)

Vue.use(CreateAPI, {
  componentPrefix: 'cube-'
  apiPrefix: '$create-'
})

import Dialog from './components/dialog.vue'

Vue.createAPI(Dialog, true)

Dialog.$create({
  $props: {
    title: 'Hello',
    content: 'I am from pure JS'
  }
}).show()

this.$createDialog({
  $props: {
    title: 'Hello',
    content: 'I am from a vue component'
  }
}).show()
```

引入 `vue-create-api` 插件，安装插件时，可以设置 `componentPrefix` 和 `apiPrefix` 两个参数，这里会在 Vue 构造器下添加一个 `createAPI` 方法。引入 Dialog 组件，调用 `createAPI` 生产对应 `API`，并挂载到 `Vue.prototype` 和 `Dialog` 对象上。之后可以在 vue 组件中通过 `this` 调用，或者在 js 文件中 `$create` 创建并使用。