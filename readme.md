# 崔锐 | Part 3 | 模块二

## 一、简答题

### 1、请简述 Vue 首次渲染的过程。

（1）Vue 的初始化，初始化了实例成员和静态成员。

（2）初始化结束后，调用了 Vue的构造函数，`new Vue()`。在构造函数中，调用 了 `this._init()` 方法（相当于 整个Vue的入口）。然后，这个 init 方法中，调用了 `$mount()` 方法。其中，有两种 `$mount()` ：

- 一种是，以 `src/platforms/web/entry-runtime-with-compiler.js` 入口文件开始，核心作用是把模板编译成函数。 首先查看是否有 `render` 选项，如果没有，就会查看 `template` 选项。如果 `template` 也没有 ，就会把 `el` 中的内容 作为模板，并把这个模板编译成`render` 函数，此处使用了 `compileToFunction()` 生成的 `render` 函数。以上 结束后，执行  `options.render = render`。
- 另一种是 ，接着 上一种的运行后 ，调用在 `src/platforms/web/runtime/index.js` 中的 $mount。这时候，重新获得 el。（运行时版本，是 不会 执行`src/platforms/web/entry-runtime-with-compiler.js` 入口的，因此需要重新获得 el。）

（3）接下来，调用 `mountComponent(this, el)` 。在 `src/core/instance/lifecycle.js` 中定义。先判断是否有 `render` 选项，如果没有，查看模板 `template`。这里有一点需要注意，如果是开发环境，没有render但有template，会发出警告（如果使用运行版本 Vue，运行时版本不支持编译器）。之后，触发了 `beforeMount` （挂载前）。然后，“定义”了 `updateComponent` ，此函数中，调用了 `vm._update(vm._render...)`（虚拟 DOM转成真实 DOM，并挂载到页面）和  （生成虚拟 DOM）函数。

- `vm._render()` 创建 虚拟 DOM `VNode`。调用  `render.call(vm_renderProxy,  vm.$createElement)`，调用了 用户传入的 `render()`（实例化时），或者编译 template 生成的 `render()`，最终把形成的  `VNode` 返回 。
- `vm._update(vnode,...)` 。调用了 把虚拟 dom 转换成 真实 dom 的方法 `vm.__patch__(vm.$el.vnode)`，挂载到页面上。并把生成的 真实 dom  设置到  `vm.$el`  中 。

（4）创建 `watcher` 实例。创建中，传入了 `updateComponent` 函数，所以这个 `updateComponent` 函数是在 watcher  中调用的。之后，调用了 `watcher`  中的 `get()` 方法。接着，触发了钩子函数中 的  `mounted` （挂载结束），最终返回 Vue 实例 `return vm`。另外 ，对于 watcher：

创建完 watcher 后，会调用一次 get 方法。在 get 方法中，调用 `updateComponent` ，从而调用 其中的  `vm._render()` 和 `vm._update(vm._render...)` 方法。

### 2、请简述 Vue 响应式原理。

（1）Vue 中的 init 方法开始的，然后出发 `initState()` 初始化Vue state状态；在 `initState()` 中调用 `initData()`， data属性注册到Vue 实力上；调用`observe()` 把 data 对象转换成 响应式对象（响应式入口）。

（2）`observe(value)` .  位置是 `src/core/observer/index.js`。判断接收到的参数value：判断value 事发后是对象，如果不是 对象直接返回；判断 value 对象是否有 `__ob__`，也就是看之前是否做过了响应式的处理，如果有直接返回，如果没有，就创建 observer 对象，之后返回  observer 对象。在创建 observer 类时，在构造函数中，给 value 对象定义不可枚举的  `__ob__` 属性，并把当前的 observer 对象记录 到 __ob__  中。之后，数组的响应式处理（pop, push, sort 等），对象的响应式处理（walk 方法，遍历 每个属性，对每个属性调用 `defineReactive`）。

（3）`defineReactive`。为每一个属性 创建 dep 对象，手机依赖。如果当前属性是对象，也要 把他 转换成响应式，调用 observe。其次，定义 getter 和 setter。`getter` 就是收集依赖，并返回属性值。`setter` 就是保存新值，如果新值是对象，调用 observe，派发更新 （发送通知），调用 `dep.notify()`。

（4）依赖收集过程。在 watcher 对象的 get 方法 中调用 `pushTarget` 记录 `Dep.target` 属性。访问 data 中的 成员 的时候收集依赖，`defineReactive` 的 `getter`  中 收集依赖。把属性对应的 watcher 对象添加到 dep 的 subs 数组中。给 `childOb` 收集依赖，目的是子对象添加和删除成员时发送通知。

（5）`watcher`。`dep.notify()`  在调用  watcher 对象的 `update()` 方法，其中 调用 queueWatcher() 判断 watcher  是否被 处理 ，如果没有的话添加到 queue  队列中 ，并调用 `flushSchedulerQueue()`。`flushSchedulerQueue()` 会触发 `beforeUpdate` 钩子函数。调用 `watcher.run()`，过程是 `run()⇒ get()⇒ getter() ⇒ updateComponent`，渲染 watcher，这时候数据已经更新到视图 上 ，在页面可以看见数据。然后就是清理过程，清空上一次的依赖，触发 `actived` 钩子函数，触发 `updated` 钩子函数。

### 3、请简述虚拟 DOM 中 Key 的作用和好处。

给节点设置 key 属性，可以跟踪节点的位置。

设置 key 比不 用key的dom 操作要少。

### 4、请简述 Vue 中模板编译的过程。

（1）模板编译的入口函数是 `compileToFunctions(template,...)` 。首先，从缓存中加载编译好的渲染函数 `render`，如果缓存中没有，调用 `compile(template, options)`。

（2）`compile(template, options)` 函数，合并 选项。先合并 options 选项 ，然后把模板和合并好的选项 传入  `baseCompile(template.trim(), finalOptions)` 编译模板。

（3）`baseCompile(template.trim(), finalOptions)`，模板编译核心三件事。

- `parse()` 先把 template 转换成 AST tree，抽象语法树。
- `optimize()` 对抽象语法树进行优化。标记 AST tree 中的静态根节点 sub trees。检测到静态子树，设置为静态，不需要在每次 重新渲染 的 时候重新生成节点。静态根节点不需要每次都重绘，patch 阶段会跳过根节点。
- `generate()`   最后把 AST tree 转换成 js 代码。

（4）回到 `compileToFunctions(template,...)` 继续把上一步中 生成的字符串形势 js 代码转换为函数，调用 `createFunction()`。`render` 和 `staticRenderFns` 初始化完毕，挂载到 Vue 实例的 options 对应的属性中。