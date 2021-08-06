我们在上一章[vue2源码解析(一)](https://mp.weixin.qq.com/s/-WKRZNRgEpYsppnbE5jjFQ)简单介绍了Vue的执行流程，本章我们创建一个例子来看一下挂载前的处理流程。
还是之前的编码环境下，我们在examples目录下新建一个my-test目录,然后创建一个init.html
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>test Vue</title>
  <script src="../../dist/vue.js"></script>
</head>
<body>
  <div id="app" @click="add">{{counter}}</div>
</body>
<script>
  const app = new Vue({
    el: '#app',
    data: function() {
      return {
        counter: 1
      }
    },
    methods: {
      add() {
        this.counter++ 
      }
    }
  }).$mount()
</script>
</html>
```
我们通过这个例子来看下Vue的初始化的流程，首先new了一个Vue实例，那我们通过源码可以定位到instance目录下的index.js中，可以找到Vue的构造方法

<font size=3 color=orange>src\core\instance\index.js</font>
```
...
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
...
```
构造方法中触发了一个_init方法，此时我们发现这个_init方法有点难找，但是同时可以看到构造方法下面又多个Mixin的方法，并且都是通过传入Vue作为参数。

我们先来看看initMixin之外的其他几个Mixin做了哪些事情
+ stateMixin： 给Vue原型对象设置属性\$data、\$props、\$set、\$del
+ eventsMixin： 给Vue原型对象设置属性\$on、\$once、\$off、\$emit
+ lifecycleMixin： 给Vue原型对象设置属性\_update、\$forceUpdate、\$destroy、\$emit
+ renderMixin： 给Vue原型对象设置属性\$nextTick、\_render

由此可见这些Mixin都是起到丰富Vue的功能的作用，这些原型对象上的方法或者属性大多是我们在日常工作中用过的，这里就不贴代码细讲，同学们可以逐个去过一遍这部分代码。

然后我们重点分析一下initMixin做了哪些工作。在initMixin中就可以看到给Vue的原型对象设置了_init,构造方法正是触发了这个方法。

<font size=3 color=orange>src\core\instance\init.js</font>

```
  Vue.prototype._init = function (options?: Object) {
    // merge options 第一步工作合并选项
    if (options && options._isComponent) {
      ...
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    // 第二步工作是设置代理
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    // 接下来是一系列处理options的工作
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }
    // 最后触发$mount方法执行挂载的操作
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
```
<font size=2 color=green>合并选项</font>
处理组件实例与父组件实例或者根实例的选项合并，还有我们用到mixin方式传入的选项，需要先进行选项的合并，然后将合并后的选项赋值给实例的$options属性

<font size=2 color=green>设置代理</font>
这一步不要理解为响应式的处理，很多同学看到Proxy就以为是响应式的处理。那这里设置的代理是为什么的，我们回到init.html的例子中
```
...
add() {
  this.counter++ 
}
...
```
我们都知道add方法中这个this是指向的Vue实例，那么为什么这里可以直接this.counter这么拿到设置在data属性里面的counter呢，这里就是这一步代理的作用，这里将选项里设置的data里面的属性都代理到Vue实例上面。

<font size=2 color=green>处理options</font>
+ initLifecycle(vm)
定义vm：$parent、\$root、\$children、\$refs等
+ initEvents(vm)
初始化事件：_events、_hasHookEvent，如果有父组件传入的事件更新到本组件上
+ initRender(vm)
初始化render相关的属性，并且给\$attrs和\$listeners属性做响应式处理
+ callHook(vm, 'beforeCreate')
触发beforeCreate钩子函数
+ initInjections(vm) // resolve injections before data/props
处理Injections注入的属性
+ initState(vm)
依次处理props、methods、data、computed、watch等选项
+ initProvide(vm) // resolve provide after data/props
处理Provide提供给后代组件的属性
+ callHook(vm, 'created')
触发created钩子函数

<font size=2 color=green>触发$mount方法</font>
接下来就开始进入编译和挂载的阶段了，这部分我们下一章再细讲

~~
感谢观看，未完待续......