#####1、请简述 Vue 首次渲染的过程。

答：
1）在渲染之前，Vue 会先初始化，初始化实例成员和静态成员。
2）当初始化完成后会调用 Vue 的构造函数(new Vue())，在构造函数中调用了\_init()方法。
这个方法相当于整个 Vue 的入口，在\_init()方法中调用了 vm.$mount() （这里的vm.$mount()是 src\platforms\web\entry-runtime-with-compiler.js 中的$mount()）。
      这个方法的核心作用是把模板编译成render函数，他会先判断是否传入了render选项， 如果没有则会去获取template选项，如果template选项也没有，则把el中的内容作为模板，然后把模板编译成render函数。
      他是通过compileToFunction()这个函数编译成render()渲染函数，当把render函数编译好之后，他会把render函数存入到options.render中
3）会调用src\platforms\web\runtime\index.js中的$mount 方法，在这个方法中会重新获取 el，因为如果是运行时版本的话是不会执行 entry-runtime-with-compiler.js 这个入口的，所以在这个方法中重新获取了 el。
4）接下来调用 mountComponent（this,el），这个方法在 src\core\instance\lifecycle.js 这个文件中定义的，会先判断是否是有 render 选项，如果没有 render 选项但是传入了模板而且是开发环境的话会发送一个警告，这个判断的目的是，如果当前是运行时版本的 vue，而且没有传入 render，但是传入了模板，那此时发送一个警告，告诉运行时版本不支持编译器。
5）触发 beforeMount 这个钩子函数，
6）定义了 updateComponent（）函数，此处仅仅是定义了这个方法， 在这个方法中调用了 vm.\_update()方法和 vm.\_render()方法。\_render 方法的作用是生成虚拟 dom， \_update()方法的作用是讲虚拟 dom 转换成真实 dom，并且挂载到页面上来。
7）创建 watcher 对象，在创建 watcher 对象的时候传入了 updateComponent()这个函数， updateComponent()这个函数最终是在 watcher 内部调用的，然后在 watcher 里面会调用 get()方法。
8）当 watcher 对象创建完毕后会调用 mounted 钩子函数。
9）最终返回 vue 实例。
10） 当创建完 watcher 后会调用一次 get 方法，在 get 方法中会调用 updateComponent()方法，在这个方法中会调用 vm.\_render 和 vm.\_update。
11）在 render 中会调用用户传入的 render()或者 template 编译生成的 render()，最终返回 VNode。
12）vm.\_update()，在这个方法中调用了 vm.**patch**(vm,$el,vnode)这个方法，这个方法就是把虚拟dom转化为真实dom并且挂载到页面上来，他会把生成的真实dom设置到vm.$el 中

1，首先初始化实例成员和静态成员.
2，调用 Vue 的构造函数，Vue 构造函数中\_init()方法调用 vm.$mount()函数，通过这个函数来生成render函数。
3，创建watcher对象，监听数据变化。
4，watcher创建后会调用get()方法中的updateComponent()方法，在这个方法中会调用vm._render()和vm._update()方法来生成VNode，并把虚拟dom转化为真实dom。
5，把真实dom设置到$el 中。

new Vue()-->\_init()-->vm.$mount()-->compileToFunction()-->mountComponent()-->触发 beforemount-->updateComponent()-->watch-->vm.\_render-->vm.\_update.

#####2、请简述 Vue 响应式原理。

答：

1）是从 Vue 实例中\_init 方法中开始的。

2）在\_init()方法中先执行 InitState()，初始化 Vue 实例的状态。

3）在 InitState()中调用了 initData()，这个方法把 data 数据注入到 Vue 实例上，并调用 observe()方法，把 data 对象转换成响应式的对象。

4）observe(value)就是响应式的入口，他接收一个参数 value 就是要进行响应式处理的对象。首先先判断 value 是否是对象，如果不是直接返回，再判断 value 是否有**ob**属性， 如果有说明这个数据之前做过响应式的处理，所以直接返回，如果没有的话，要为这个对象创建 observer 对象，最后返回 observer 对象 。

5）在创建 observe 对象过程中做了什么事情呢？首先会给当前 value 对象定义一个不可枚举的**ob**属性，并把当前 observer 对象记录到**ob**里边来，然后再进行数组的响应式处理和对象的响应式处理。
数组的响应式处理其实就是设置数组的那几个特殊的方法，比如说 push，pop，sort 等等，这些方法会改变原数组，所以这些方法被调用时候我们要发送通知， 发送通知的时候，是找到数组对象对应的 ob，也就是 observer 对象，在找到 observer 中的 dep，调用 dep 的 notify 方法。
更改完了数组的这些特殊的方法之后，遍历数组中的每一个成员， 对每一个成员再去调用 observe()，如果这个成员也是对象的话会对这个对象设置成响应式的对象。
6）对象的响应式处理，就是如果 value 是对象的话，那此时会调用 walk()方法，walk()方法就是遍历这个对象的所有的属性，对每一个属性调用 defineReactive。
在 defineReactive 中，会对每一个属性创建 dep 对象，让 dep 去收集依赖，如果当前属性的值是对象的话，会调用 observe，要把这个对象也转化为响应式的对象。

7）
在 defineReactive 中首先创建了一个 Dep 对象， 这个 Dep 对象是为当前这个属性收集依赖，也就是收集观察当前这个属性的所有 watcher。
然后通过 getOwnPropertyDescriptor(obj,key)来获取当前属性的属性描述符，判断属性描述符里的 configurable 是否是 false，如果是 false，说明当前属性是不可配置的，说明当前这个属性不能被 delete 删除，还不可以通过 Object.defineProperty 重新定义， 因为接下来需要通过 Object.defineProperty 重新给这个属性定义属性描述符，所以他不可配置则直接返回。

     在defineReactive中最核心的是定义getter和setter。
     在getter里收集依赖，收集依赖的时候要为每一个属性收集依赖，如果这个属性的值是对象，那么她也要为这个子对象收集依赖，在getter里面最终返回该属性的值。
     在setter里面，首先先把新值保存下来，如果新值是对象的话也要调用observe，也要把新设置的对象也转化为响应式的对象。在setter里面数据发生了变化，所以要发送通知，发送通知其实就是调用dep.notify()方法。

8）收集依赖的过程，收集依赖的时候首先执行 watcher 对象里的 get()方法。
在 get()方法中会调用 pushTarget，在 pushTarget 中会把当前的 watcher 对象记录到 Dep.target 属性中，然后在访问 data 成员属性的时候去收集依赖，在这个时候， 当我们访问这个属性的值得时候就会触发 defineReactive 中的 getter，在 getter 中回去收集依赖， 他会把属性对应的 watcher，添加到 dep 的 subs 数组中， 也就是为属性收集依赖，
如果这个属性的值也是对象，那此时要创建一个 childOb 对象，要为我们这个子对象收集依赖，目的是将来子对象发生变化的时候，那么可以发送通知。

9）看一下 watcher，当数据发生变化的时候， 会调用 dep.notify()发送通知，他会调用 wacher 对象的 update()方法。
在 wacher 的 update()方法中会 queueWatcher()这个函数，在 queueWatcher 里面会判断 watcher 是否被处理了，如果 watcher 对象没有被处理，那么他会添加到 queue 队列中， 并且调用 flushSchedulerQueue()，刷新任务队列函数。
在 flushSchedulerQueue()函数中会触发 beforeUpdate 钩子函数，然后调用 watcher.run()这个方法。
在 watcher.run()这个方法中，去调用 watcher 的 get 方法， 去调用 getter，getter 里边存储的就是 updateComponent，这是针对渲染 watcher 来说的。
watcher.run()运行完成之后，其实就把数据更新到了视图上， 在页面上可以看到最新的数据。

10）接下来进行清理的工作， 他会清空上一次的依赖，他会重置 watcher 中的一些状态。

11）接下来触发 actived 钩子函数

12）最后触发 updated 钩子函数。

#####3、请简述虚拟 DOM 中 Key 的作用和好处。

答：
key 的作用和好处是给虚拟 Dom（Vnode）提供索引，进行删除、添加等操作时更加快捷。通过设置不同的 key，可以完整地触发组件的生命周期钩子和触发过渡。可以帮助我们快速对比两个虚拟 dom 对象，找到虚拟 dom 对象被修改的元素，然后仅仅替换掉被修改的元素，然后再生成新的真实 dom。

#####4、请简述 Vue 中模板编译的过程。

答：

模板编译的入口函数 compileToFunctions(),首先从缓存中加载编译好的 render 函数，如果缓存中没有的话则调用 compile()函数开始编译，
在 compile()函数中首先合并 options 选项，然后调用 baseCompile()函数编译模板,compile()函数的核心是合并选项，真正处理是在 baseCompile()函数中完成的， 把模板和合并好的选项传递给 baseCompile()函数，他里边完成模板编译核心的三件事情 1）首先模板字符串转换为 AST 对象也就是抽象语法树 2）然后对抽象语法树 AST 进行优化，标记 AST 中的所有静态根节点，静态根节点不需要每次都重绘，patch 阶段会跳过静态根节点 3）最后把优化过的 AST 对象转换成字符串形式的代码。
当 compile 执行完毕后，最后会回到编译的入口函数 compileToFunctions()，它里边会继续把字符串形式的代码转换成函数的形式，通过调用 createFunction()，当 render 和 staticRenderFns 创建完毕后都会被挂在到 Vue 实例中的 options 选项对应的属性上，
在模板编译的过程中可以标记静态根节点，对静态根节点进行优化处理， 重新渲染的时候不需要从新处理静态根节点，因为他的内容不会发送改变，另外在模板中不要写过多的无意义的空格和换行，否则生成的 AST 对象会保留这些空白和换行，他们都会被存储到内存中，而这些空格和换行对浏览器渲染来说没有任何意义。
