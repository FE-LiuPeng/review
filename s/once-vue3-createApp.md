### <center>一更新文--Vue3的createApp</center>

> 这 拖了一年了， 差不多全都忘了... 漂亮！（本文只做简单的概述）

从命令行入手吧.. 
1. npm run dev 
2. 走进了`scripts/dev.js` 
3. 开始解析参数， 然后走进rollup把参数搞里头，
4. 进入了`rollup.config.js`这里头有有入口文件， 分为runtime和默认版本
   ```js
    // rollup.config.js 120多行
    // format 这玩意去第3步的参数里面找。
      let entryFile = /runtime$/.test(format) ? `src/runtime.ts` : `src/index.ts`
   ```
5. 好，这么轻易的就找到了入口，那然呢？然后就是我们自己个儿调用createApp了
6. createApp入口 `runtime-dom/src/index.ts`56行
    ```js
        export const createApp = ((...args) => {
        console.log('初始化vue--- createApp')
        const app = ensureRenderer().createApp(...args)
        const { mount } = app// mount 单拉出来去储存到堆里面 
        app.mount = (containerOrSelector: Element | ShadowRoot | string): any => {
            const container = normalizeContainer(containerOrSelector)
            if (!container) return

            const component = app._component; 
            if (!isFunction(component) && !component.render && !component.template) {
            component.template = container.innerHTML
            }

            container.innerHTML = ''
            const proxy = mount(container, false, container instanceof SVGElement)
            if (container instanceof Element) {
            container.removeAttribute('v-cloak')
            container.setAttribute('data-v-app', '')
            }
            return proxy
        }

        return app
        })
   ```
   1. 额 没了 挖哈哈哈哈哈， 第一步初始化实例，然后获取实例
      - 第一步创建渲染函数，并初始化一些必要的方法出来， 比如对于`dom的func`，虚拟`dom的func`，以及`diff，patch`这些都初始一下。然后全都搞完之后返回了一个对象，其中有一个我们耳熟能详的`createApp`
      - 第二步就要进入到`createApp`里面去，一探究竟，在`createAppApi`里面主要是对组件的初始化， 像是`mixin`、`component`、`version`、`use`、`directive`、`unmount`等等。
      - 这时候已经准备好了， 然后返回应用(app)。
   2. 先把`mount`拿出来，然后重写
      - 调用`normalizeContainer`获取根元素容器；
      - 判断`template`，获取需要渲染的模板；（这里就可以看到我们伟大的vue都支持啥玩意）
      - 把容器的`innerHTML`置空；
      - 开始挂载函数的重写
      - 删除`v-cloak属性，添加data-v-app属性`；
      - 返回`mount`后的代理； 
   3. 返回实例
   4. 执行createApp，并执行mount
      - 第一步先要执行`runtime-dom`里面的mount， 然后走上面的哪些步骤， 
      - 开始执行 `createAppApi`里面的mount
      - 进行vnode的创建、patch替换、render渲染

**好， 进入主题**
#### createRenderer（ensureRenderer）
```js
// options 参数位置： 
// 1. runtime-dom/src/nodeOps.ts -- nodeOps
// 2. runtime-dom/src/patchProp.ts -- forcePatchProp 和 patchProp
func baseCreateRenderer(options: 对dom和虚拟dom的增删改查的一系列方法 ) {
    // 为了方便查看 简单些总结下
    // 1. 函数解构，声明
    // 2. 对于虚拟dom的一些更替的函数声明
    // 3. 挂在子vnode的函数
    // 4. 有key的时候对比的diff
    // 5. 没有key的时候对比的函数
    // 6. 等等等等 一堆函数声明
    // diff “核心”的算法在这个文件里面最后一个函数 getSequence

    return {
        render,
        hydrate,
        createApp: createAppAPI(render, hydrate)
    }
}
```

**好， 时间来到了`baseCreateRenderer`返回的`createApp`函数， 那我们继续分解**
#### createAppAPI
> 额 先不要看那个混合的函数。 
```js
 function createAppAPI(render) {
  return function createApp(rootComponent, rootProps = null) {
    if (rootProps != null && !isObject(rootProps)) {
      rootProps = null
    }

    const context = createAppContext()
    const installedPlugins = new Set()

    let isMounted = false

    const app: App = (context.app = {
      _uid: uid++,
      _component: rootComponent,
      _props: rootProps,
      _container: null,
      _context: context,
      _instance: null,

      version,

      get config() {
        return context.config
      },

      set config(v) {},

      use(plugin: Plugin, ...options: any[]) {},

      mixin(mixin: ComponentOptions) {},

      component(name: string, component?: Component): any {},

      directive(name: string, directive?: Directive) {},
      // 挂在Vdom 开始初始化renderTime patch
      mount(
        rootContainer: HostElement,
        isHydrate?: boolean,
        isSVG?: boolean
      ): any {
        if (!isMounted) {
          const vnode = createVNode(
            rootComponent as ConcreteComponent,
            rootProps
          )
          vnode.appContext = context

            render(vnode, rootContainer, isSVG)
          isMounted = true
          app._container = rootContainer
          ;(rootContainer as any).__vue_app__ = app
          return vnode.component!.proxy
        } 
      },

      unmount() {
        if (isMounted) {
          render(null, app._container)
          delete app._container.__vue_app__
        } 
      },

      provide(key, value) {
        context.provides[key as string] = value
        return app
      }
    })
    return app
  }
}
```

诶呀 这里面我咋感觉这么熟悉呢， 这就是我们初始话应用的内部信息（也大概包括组建的）。
这玩意搞完了就要进入下一个环节了， 就是挂载（执行mount函数）;

#### mount 
最核心(繁琐)的操作都在mount里面,这里面包括Vnode，render，patch等等所有的核心功能。咱们先理一下mount的调用流程。

第一步：调用runtime-dom下面index.ts里面的mount函数；
第二步：调用runtime-core下面apiCreateApp里面的mount函数；


完结。
