---
title: 网页实现把文本复制到粘贴板
date: 2017-12-27 14:28:33
categories:
- 学习
tags:
- 工作
- vue
---
为了写一个运营用得小工具，需要把一段文本复制下来方便粘贴，整个过程学到了网页中复制文本所需API和一些封装好的轮子，遂积累下来分享。
<!-- more -->
<!-- excerpt -->
## 思路整理
有一个按钮可以触发以下逻辑：
- 将生成的文本自动复制到剪切板上 结果要有友好提示
- 复制成功后可以粘贴到任何地方

一开始我以为有通用的接口，一番调研之后发现有以下几种东东：
1. `document.execCommand("copy")`这个用于在要复制的文本处于被选中状态时使用
2. `window.clipboardData.setData("Text", clipBoardContent)`为挂载在window上的API 据说兼容性不好
3. 结合`ZeroClipboard.js`这个插件实现兼容性比较好的复制粘贴，通过`new ZeroClipboard.client()`来调用各个方法
4. `github`上的库[`clipboard.js`](https://github.com/zenorocha/clipboard.js),通过`new Clipboard('.btn')`这种语法实现访问剪切板的操作,兼容性最好

*有一些具体的代码示例参考CSDN中的[内容](http://blog.csdn.net/fanhu6816/article/details/52809385),不过要考虑是否过时*

由于是在vue框架基础上进行开发，最后选择了上面提到的第四种方法，完全抛弃了flash的hack实现(第三种)，并且有着强大的兼容性，`github`上的star数量已经说明了一切，本来考虑是自己封装个指令来用，后来发现[`vue-clipboard2`](https://github.com/Inndy/vue-clipboard2)这个插件已经实现了封装，按照绝不重复造轮子的原则，直接在项目里用了起来。

## 原理剖析
`clipboard.js`的核心原理是虚拟了一个不可见的选区并利用复制的API来实现文本复制，因此最起码需要动态创造的页面元素有可以有被选中的属性。
为了方便学习vue自定义指令的写法，现在把vue-clipboard2的代码摘抄过来加以学习：
```javascript
var Clipboard = require('clipboard')

var VueClipboard = {
  install: function (Vue) {
    Vue.prototype.$copyText = function (text) { 
      // 这个是非触发式的调用api(this.$copyText)
      return new Promise(function (resolve, reject) {
        var fake_el = document.createElement('button');
        var clipboard = new Clipboard(fake_el, {
          text: function () { return text },
          action: function () { return 'copy' }
        });
        clipboard.on('success', function (e) {
          clipboard.destroy();
          resolve(e);
        });
        clipboard.on('error', function (e) {
          clipboard.destroy();
          reject(e);
        });
        fake_el.click();
      });
    };
    // 自定义指令时要注意vue自定义指令的生命周期
    Vue.directive('clipboard', {
      bind: function (el, binding, vnode) {
        if(binding.arg === 'success') {
          el._v_clipboard_success = binding.value
        } else if(binding.arg === 'error') {
          el._v_clipboard_error = binding.value
        } else {
          var clipboard = new Clipboard(el, {
            text: function () { return binding.value },
            action: function () { return binding.arg === 'cut' ? 'cut' : 'copy' }
          })
          clipboard.on('success', function (e) {
            var callback = el._v_clipboard_success
            callback && callback(e)
          })
          clipboard.on('error', function (e) {
            var callback = el._v_clipboard_error
            callback && callback(e)
          })
          el._v_clipboard = clipboard
        }
      },
      update: function (el, binding) {
        if(binding.arg === 'success') {
          el._v_clipboard_success = binding.value
        } else if(binding.arg === 'error') {
          el._v_clipboard_error = binding.value
        } else {
          el._v_clipboard.text = function () { return binding.value }
          el._v_clipboard.action = function () { return binding.arg === 'cut' ? 'cut' : 'copy' }
        }
      },
      unbind: function (el, binding) {
        if(binding.arg === 'success') {
          delete el._v_clipboard_success
        } else if(binding.arg === 'error') {
          delete el._v_clipboard_error
        } else {
          el._v_clipboard.destroy()
          delete el._v_clipboard
        }
      }
    })
  }
}

// 以下是把这个模块输出的写法，兼容了amd/cmd
if (typeof exports == "object") {
  module.exports = VueClipboard
} else if (typeof define == "function" && define.amd) {
  define([], function() {
    return VueClipboard
  })
}
```

## 插件使用
如果对`vue-directive`比较熟悉的话，这些看懂都比较容易，核心就是`new Clipboard()`的调用，看完之后在vue中的用法自然很容易。定义的指令名为`v-clipboard`。并且可以传入`v-clipboard:cut`/`v-clipboard:copy`/`v-clipboard:success`/`v-clipboard:error`等参数及回调函数。
```html
<el-button v-clipboard:copy="finalLink" v-clipboard:success="copySuccess" v-clipboard:error="copyFail" type="primary">复制链接</el-button>
```

```javascript
import Vue from 'vue'
import VueClipboard from 'vue-clipboard2'
Vue.use(VueClipboard) //使用自定义指令的写法
...
copySuccess() {
    this.$message({
        message: '链接已复制，请粘贴',
        type: 'success'
    })
},
copyFail() {
    this.$message.error('复制失败，请手动操作')
},
```
在element-ui基础上开发而成，所以成功和失败的回调都直接用了element的API。相信还是比较容易看懂的哈~
