### <center>一更新文--前端框架库</center>

#### vue2/3，react diff对比
- react-diff
  1. 实现原理
    > React的思路是递增法。通过对比新的列表中的节点，在原本的列表中的位置是否是递增，来判断当前节点是否需要移动。
- vue2-diff （双端比较）
    > 所谓双端比较就是新列表和旧列表两个列表的头与尾互相对比，，在对比的过程中指针会逐渐向内靠拢，直到某一个列表的节点全部遍历过，对比停止。
    数据更新后会出发watcher的毁掉函数vm._update(vm._render())去更新视图， render的时候就会生成新的vnode，在更新（update）的时候会出发patch。

    **patch**
    主要干的事儿：  
     - vnode 不存在，oldVnode 存在，就删掉 oldVnode
     - vnode 存在，oldVnode 不存在，就创建 vnode
     - 两个都存在的话，通过 sameVnode 函数(后面有详解)对比是不是同一节点
       - 如果是同一节点的话，通过 patchVnode 进行后续对比节点文本变化或子节点变化
       - 如果不是同一节点，就把 vnode 挂载到 oldVnode 的父元素下
         - 如果组件的根节点被替换，就遍历更新父节点，然后删掉旧的节点
         - 如果是服务端渲染就用 hydrating 把 oldVnode 和真实 DOM 混合
    ```js
        // 两个判断函数
        function isUndef (v: any): boolean %checks {
        return v === undefined || v === null
        }
        function isDef (v: any): boolean %checks {
        return v !== undefined && v !== null
        }
        return function patch (oldVnode, vnode, hydrating, removeOnly) {
            // 如果新的 vnode 不存在，但是 oldVnode 存在
            if (isUndef(vnode)) {
            // 如果 oldVnode 存在，调用 oldVnode 的组件卸载钩子 destroy
            if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
            return
            }

            let isInitialPatch = false
            const insertedVnodeQueue = []
            
            // 如果 oldVnode 不存在的话，新的 vnode 是肯定存在的，比如首次渲染的时候
            if (isUndef(oldVnode)) {
            isInitialPatch = true
            // 就创建新的 vnode
            createElm(vnode, insertedVnodeQueue)
            } else {
            // 剩下的都是新的 vnode 和 oldVnode 都存在的话
            
            // 是不是元素节点
            const isRealElement = isDef(oldVnode.nodeType)
            // 是元素节点 && 通过 sameVnode 对比是不是同一个节点 (函数后面有详解)
            if (!isRealElement && sameVnode(oldVnode, vnode)) {
                // 如果是 就用 patchVnode 进行后续对比 (函数后面有详解)
                patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
            } else {
                // 如果不是同一元素节点的话
                if (isRealElement) {
                // const SSR_ATTR = 'data-server-rendered'
                // 如果是元素节点 并且有 'data-server-rendered' 这个属性
                if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
                    // 就是服务端渲染的，删掉这个属性
                    oldVnode.removeAttribute(SSR_ATTR)
                    hydrating = true
                }
                // 这个判断里是服务端渲染的处理逻辑，就是混合
                if (isTrue(hydrating)) {
                    if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
                    invokeInsertHook(vnode, insertedVnodeQueue, true)
                    return oldVnode
                    } else if (process.env.NODE_ENV !== 'production') {
                    warn('这是一段很长的警告信息')
                    }
                }
                // function emptyNodeAt (elm) {
                //    return new VNode(nodeOps.tagName(elm).toLowerCase(), {}, [], undefined, elm)
                //  }
                // 如果不是服务端渲染的，或者混合失败，就创建一个空的注释节点替换 oldVnode
                oldVnode = emptyNodeAt(oldVnode)
                }
                
                // 拿到 oldVnode 的父节点
                const oldElm = oldVnode.elm
                const parentElm = nodeOps.parentNode(oldElm)
                
                // 根据新的 vnode 创建一个 DOM 节点，挂载到父节点上
                createElm(
                vnode,
                insertedVnodeQueue,
                oldElm._leaveCb ? null : parentElm,
                nodeOps.nextSibling(oldElm)
                )
                
                // 如果新的 vnode 的根节点存在，就是说根节点被修改了，就需要遍历更新父节点
                if (isDef(vnode.parent)) {
                let ancestor = vnode.parent
                const patchable = isPatchable(vnode)
                // 递归更新父节点下的元素
                while (ancestor) {
                    // 卸载老根节点下的全部组件
                    for (let i = 0; i < cbs.destroy.length; ++i) {
                    cbs.destroy[i](ancestor)
                    }
                    // 替换现有元素
                    ancestor.elm = vnode.elm
                    if (patchable) {
                    for (let i = 0; i < cbs.create.length; ++i) {
                        cbs.create[i](emptyNode, ancestor)
                    }
                    const insert = ancestor.data.hook.insert
                    if (insert.merged) {
                        for (let i = 1; i < insert.fns.length; i++) {
                        insert.fns[i]()
                        }
                    }
                    } else {
                    registerRef(ancestor)
                    }
                    // 更新父节点
                    ancestor = ancestor.parent
                }
                }
                // 如果旧节点还存在，就删掉旧节点
                if (isDef(parentElm)) {
                removeVnodes([oldVnode], 0, 0)
                } else if (isDef(oldVnode.tag)) {
                // 否则直接卸载 oldVnode
                invokeDestroyHook(oldVnode)
                }
            }
            }
            // 返回更新后的节点
            invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
            return vnode.elm
        }

    ```
    **sameVnode**  判断是否相同节点
    ```js
    function sameVnode (a, b) {
            return (
                a.key === b.key && (
                (
                    a.tag === b.tag &&// 标签是不是一样
                    a.isComment === b.isComment &&// 是不是注释节点
                    isDef(a.data) === isDef(b.data) && // // 内容数据是不是一样
                    sameInputType(a, b)// 判断 input 的 type 是不是一样
                )
                )
            )
        }
    ```
    **patchVnode**（这个是在新的 vnode 和 oldVnode 是同一节点的情况下，才会执行的函数，主要是对比节点文本变化或子节点变化）  
    还是先介绍一下主要流程，再看源码吧，流程是这样的：
    - 如果 oldVnode 和 vnode 的引用地址是一样的，就表示节点没有变化，直接返回
    - 如果 oldVnode 的 isAsyncPlaceholder 存在，就跳过异步组件的检查，直接返回
    - 如果 oldVnode 和 vnode 都是静态节点，并且有一样的 key，并且 vnode 是克隆节点或者 v-once 指令控制的节点时，把 oldVnode.elm 和 oldVnode.child 都复制到 vnode 上，然后返回
    - 如果 vnode 不是文本节点也不是注释的情况下
      - 如果 vnode 和 oldVnode 都有子节点，而且子节点不一样的话，就调用 updateChildren 更新子节点
      - 如果只有 vnode 有子节点，就调用 addVnodes 创建子节点
      - 如果只有 oldVnode 有子节点，就调用 removeVnodes 删除该子节点
      - 如果 vnode 文本为 undefined，就删掉 vnode.elm 文本
    - 如果 vnode 是文本节点但是和 oldVnode 文本内容不一样，就更新文本
    ```js
        function patchVnode (
            oldVnode, // 老的虚拟 DOM 节点
            vnode, // 新的虚拟 DOM 节点
            insertedVnodeQueue, // 插入节点的队列
            ownerArray, // 节点数组
            index, // 当前节点的下标
            removeOnly // 只有在
        ) {
            // 新老节点引用地址是一样的，直接返回
            // 比如 props 没有改变的时候，子组件就不做渲染，直接复用
            if (oldVnode === vnode) return
            
            // 新的 vnode 真实的 DOM 元素
            if (isDef(vnode.elm) && isDef(ownerArray)) {
            // clone reused vnode
            vnode = ownerArray[index] = cloneVNode(vnode)
            }

            const elm = vnode.elm = oldVnode.elm
            // 如果当前节点是注释或 v-if 的，或者是异步函数，就跳过检查异步组件
            if (isTrue(oldVnode.isAsyncPlaceholder)) {
            if (isDef(vnode.asyncFactory.resolved)) {
                hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
            } else {
                vnode.isAsyncPlaceholder = true
            }
            return
            }
            // 当前节点是静态节点的时候，key 也一样，或者有 v-once 的时候，就直接赋值返回
            if (isTrue(vnode.isStatic) &&
            isTrue(oldVnode.isStatic) &&
            vnode.key === oldVnode.key &&
            (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
            ) {
            vnode.componentInstance = oldVnode.componentInstance
            return
            }
            // hook 相关的不用管
            let i
            const data = vnode.data
            if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
            i(oldVnode, vnode)
            }
            // 获取子元素列表
            const oldCh = oldVnode.children
            const ch = vnode.children
            
            if (isDef(data) && isPatchable(vnode)) {
            // 遍历调用 update 更新 oldVnode 所有属性，比如 class,style,attrs,domProps,events...
            // 这里的 update 钩子函数是 vnode 本身的钩子函数
            for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
            // 这里的 update 钩子函数是我们传过来的函数
            if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
            }
            // 如果新节点不是文本节点，也就是说有子节点
            if (isUndef(vnode.text)) {
            // 如果新老节点都有子节点
            if (isDef(oldCh) && isDef(ch)) {
                // 如果新老节点的子节点不一样，就执行 updateChildren 函数，对比子节点
                if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
            } else if (isDef(ch)) {
                // 如果新节点有子节点的话，就是说老节点没有子节点
                
                // 如果老节点文本节点，就是说没有子节点，就清空
                if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
                // 添加子节点
                addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
            } else if (isDef(oldCh)) {
                // 如果新节点没有子节点，老节点有子节点，就删除
                removeVnodes(oldCh, 0, oldCh.length - 1)
            } else if (isDef(oldVnode.text)) {
                // 如果老节点是文本节点，就清空
                nodeOps.setTextContent(elm, '')
            }
            } else if (oldVnode.text !== vnode.text) {
            // 新老节点都是文本节点，且文本不一样，就更新文本
            nodeOps.setTextContent(elm, vnode.text)
            }
            if (isDef(data)) {
            // 执行 postpatch 钩子
            if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
            }
        }
    ```
    **updateChildren**
   ` 这个是新的 vnode 和 oldVnode 都有子节点，且子节点不一样的时候进行对比子节点的函数 ` 
      1. 使用旧列表的头一个节点oldStartNode与新列表的头一个节点newStartNode对比
      2. 使用旧列表的最后一个节点oldEndNode与新列表的最后一个节点newEndNode对比
      3. 使用旧列表的头一个节点oldStartNode与新列表的最后一个节点newEndNode对比
      4. 使用旧列表的最后一个节点oldEndNode与新列表的头一个节点newStartNode对比
   旧首节点和新首节点用 sameNode 对比, 旧尾节点和新尾节点用 sameNode 对比,旧首节点和新尾节点用 sameNode 对比,旧尾节点和新首节点用 sameNode 对比，如果是相同节点，就会进行patchnode的对比，对比子节点或者文字节点...  
   如果以上逻辑都匹配不到，再把所有旧子节点的 key 做一个映射到旧节点下标的 key -> index 表，然后用新 vnode 的 key 去找出在旧节点中可以复用的位置。  
   为什么会有头对尾，尾对头的操作？因为可以快速检测出 reverse 操作，加快 Diff 效率
   ```js
        function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
            let oldStartIdx = 0 // 老 vnode 遍历的下标
            let newStartIdx = 0 // 新 vnode 遍历的下标
            let oldEndIdx = oldCh.length - 1 // 老 vnode 列表长度
            let oldStartVnode = oldCh[0] // 老 vnode 列表第一个子元素
            let oldEndVnode = oldCh[oldEndIdx] // 老 vnode 列表最后一个子元素
            let newEndIdx = newCh.length - 1 // 新 vnode 列表长度
            let newStartVnode = newCh[0] // 新 vnode 列表第一个子元素
            let newEndVnode = newCh[newEndIdx] // 新 vnode 列表最后一个子元素
            let oldKeyToIdx, idxInOld, vnodeToMove, refElm

            const canMove = !removeOnly
            
            // 循环，规则是开始指针向右移动，结束指针向左移动移动
            // 当开始和结束的指针重合的时候就结束循环
            while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
            if (isUndef(oldStartVnode)) {
                oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
            } else if (isUndef(oldEndVnode)) {
                oldEndVnode = oldCh[--oldEndIdx]
                
                // 老开始和新开始对比
            } else if (sameVnode(oldStartVnode, newStartVnode)) {
                // 是同一节点 递归调用 继续对比这两个节点的内容和子节点
                patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                // 然后把指针后移一位，从前往后依次对比
                // 比如第一次对比两个列表的[0]，然后比[1]...，后面同理
                oldStartVnode = oldCh[++oldStartIdx]
                newStartVnode = newCh[++newStartIdx]
                
                // 老结束和新结束对比
            } else if (sameVnode(oldEndVnode, newEndVnode)) {
                patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
                // 然后把指针前移一位，从后往前比
                oldEndVnode = oldCh[--oldEndIdx]
                newEndVnode = newCh[--newEndIdx]
                
                // 老开始和新结束对比
            } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
                patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
                canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
                // 老的列表从前往后取值，新的列表从后往前取值，然后对比
                oldStartVnode = oldCh[++oldStartIdx]
                newEndVnode = newCh[--newEndIdx]
                
                // 老结束和新开始对比
            } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
                patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
                // 老的列表从后往前取值，新的列表从前往后取值，然后对比
                oldEndVnode = oldCh[--oldEndIdx]
                newStartVnode = newCh[++newStartIdx]
                
                // 以上四种情况都没有命中的情况
            } else {
                if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
                // 拿到新开始的 key，在老的 children 里去找有没有某个节点有这个 key
                idxInOld = isDef(newStartVnode.key)
                ? oldKeyToIdx[newStartVnode.key]
                : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
                
                // 新的 children 里有，可是没有在老的 children 里找到对应的元素
                if (isUndef(idxInOld)) {
                /// 就创建新的元素
                createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
                } else {
                // 在老的 children 里找到了对应的元素
                vnodeToMove = oldCh[idxInOld]
                // 判断标签如果是一样的
                if (sameVnode(vnodeToMove, newStartVnode)) {
                    // 就把两个相同的节点做一个更新
                    patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                    oldCh[idxInOld] = undefined
                    canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
                } else {
                    // 如果标签是不一样的，就创建新的元素
                    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
                }
                }
                newStartVnode = newCh[++newStartIdx]
            }
            }
            // oldStartIdx > oldEndIdx 说明老的 vnode 先遍历完
            if (oldStartIdx > oldEndIdx) {
            // 就添加从 newStartIdx 到 newEndIdx 之间的节点
            refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
            addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
            
            // 否则就说明新的 vnode 先遍历完
            } else if (newStartIdx > newEndIdx) {
            // 就删除掉老的 vnode 里没有遍历的节点
            removeVnodes(oldCh, oldStartIdx, oldEndIdx)
            }
        }
   ```

- vue3-diff （最长递增子序列）
  ```js
    const s1 = '你好，世界';
    const s2 = '你，世界';
    // 不一样的是 缺少了一个'好'
  ```
  children对比， 头对头，尾对尾， 然后最长递增子序列进行移动/添加/删除  
  针对于有key的时候，进行diff算法，没有key的时候 直接去patch， （**patchKeyedChildren**，**patchUnkeyedChildren**）
  **patchUnkeyedChildren**
  ```js
    // 地址 renderer 里面 1700多行 我注视比较多， 可能乱行了
    const patchUnkeyedChildren = (
    c1: VNode[],
    c2: VNodeArrayChildren,
    container: RendererElement,
    anchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
  ) => {
    c1 = c1 || EMPTY_ARR
    c2 = c2 || EMPTY_ARR
    const oldLength = c1.length
    const newLength = c2.length
    const commonLength = Math.min(oldLength, newLength) // 交集
    let i
    for (i = 0; i < commonLength; i++) {// 遍历新旧中最短的节点，依次patch，如果不是相同节点，直接卸载
      const nextChild = (c2[i] = optimized
        ? cloneIfMounted(c2[i] as VNode)
        : normalizeVNode(c2[i]))
      patch( // 在进行递归
        c1[i],
        nextChild,
        container,
        null,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      )
    }
    //     // 新的变短了就删除， 变长了就新增
    if (oldLength > newLength) { // 删节点
      // remove old
      unmountChildren(
        c1,
        parentComponent,
        parentSuspense,
        true,
        false,
        commonLength
      )
    } else {
      // mount new // 添加节点
      mountChildren(
        c2,
        container,
        anchor,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized,
        commonLength
      )
    }
  }
  // 结论： 
    1. 比较新老children的length获取最小值 然后对于公共部分，进行从新patch工作。
    2. 如果老节点数量大于新的节点数量 ，移除多出来的节点。
    3. 如果新的节点数量大于老节点的数量，从新 mountChildren新增的节点。
  ```
  **patchKeyedChildren**
    这个就是 头对头，尾对尾， 然后最长递增子序列进行移动/添加/删除 ；这里面有啥个变量比较重要， 在源码里面都已经清晰的描述出来了， 
    `i, e1, e2`， i： 遇见不一样的节点就会停下， e1：代表老节点的不同的索引开始值， 如果新节点大于则尾部为最大值， 若小于0则为-1， e2于上述e1功能一致。
    这部分代码在 renderer里面的patchKeyedChildren代码块里面
    **前后对比**
    ```js
        let i = 0;   
        let e1 = c1.length - 1
        let e2 = c2.length - 1

        从前往后遍历，i增加. 遇到不一样的节点暂停。i 的值就很重要了。
        while (i <= e1 && i <= e2) {
            const n1 = c1[i]
            const n2 = c2[i]
            if (isSameVNodeType(n1, n2)) {
                patch（...）
            } else {
                break
            }
            i++
            }

        再从后往前遍历，原元素递减，遇到相同的也暂停。
            while (i <= e1 && i <= e2) {
            const n1 = c1[e1]
            const n2 = c2[e2]
            if (isSameVNodeType(n1, n2)) {
                patch（...）
            } else {
                break
            }
            e1--
            e2--
            }
    ```
    
    下面拿来源码里面的几行代码来对上述形式理解：
    ```js
    1. i=2, e1 = 2, e2 = 3
    // (a b) c
    // (a b) d e
    2. i = 0, e1 = 0, e2 = 1   
    // a (b c)
    // d e (b c)
    3. i = 2, e1 = 1, e2 = 2
    // a b
    // a b c
    4. i = 0, e1 = -1, e2 = 0
    // a b
    // c a b f g h
    5. i = 0, e1 = 0, e2 = -1
    // m a b
    // a b
    6. i =1, e1 = 1, e2 = 0
    // k l j
    // k j 
    7. i = 2, e1 = 4, e2 = 5
    // [i ... e1 + 1]: a b [c d e] f g
    // [i ... e2 + 1]: a b [e d c h] f g
    ```
        相同的话就去进行patch，不同的话就跳出去， 上面的i非常重要， 从上面可以看出规律: 单纯的新增元素时： i > e1, 单纯的减少元素时： i > e2. 如果 i 比e1 e2 小，那就不是单纯新增或减少了。单纯新增减少元素不会涉及到diff算法的核心算法。其他情况else才需要用到
    ```js
        if (i > e1) {
      if (i <= e2) {
        const nextPos = e2 + 1
        const anchor = nextPos < l2 ? (c2[nextPos] as VNode).el : parentAnchor
        // 创建新的 增加 
        while (i <= e2) {
          patch(
            null,
            (c2[i] = optimized
              ? cloneIfMounted(c2[i] as VNode)
              : normalizeVNode(c2[i])),
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
          i++
        }
      }
    }

    // 4. common sequence + unmount
    // 少了 就删除
    else if (i > e2) {
      while (i <= e1) {
        unmount(c1[i], parentComponent, parentSuspense, true)
        i++
      }
    }
    ```
    但是 还有最后一种情况， 也是应用diff的情况， 就是 两边一致，中间搞事儿！就在最后的else里面， 
    ```js
       1. 第一步遍历到的index，存起
       const s1 = i 
       const s2 = i 

       2. 前后对比循环后，,通过map 记录 c2 没有比较过的节点的位置角标
       const keyToNewIndexMap: Map<string | number, number> = new Map()
        /*  */
             for (i = s2; i <= e2; i++) {
                const nextChild = c2[i]
               if (nextChild.key != null) {
                 keyToNewIndexMap.set(nextChild.key, i)
               }
             }

       3.  c2 中待处理的节点数目 
       const toBePatched = e2 - s2 + 1

       4. 证明是否移动
       let moved = false 

       5. 记录节点在 c2 中的最大索引，假如节点在 c2 中的索引位置小于这个最大索引，代表有元素需要移动
       let maxNewIndexSoFar = 0 

       6. 针对未比较的C2 节点建立一个数组，每个子元素都是0 [ 0, 0, 0, 0, 0, 0] 
       初始情况，0 代表新增
       const newIndexToOldIndexMap = new Array(toBePatched)
       for (i = 0; i < toBePatched; i++) {
           newIndexToOldIndexMap[i] = 0
       }

    ```
    循环旧的节点
    ```js
        已处理的节点
        let patched = 0 
        let j 

        for (i = s1; i <= e1; i++) {
        // 如果已处理的数量大于待处理节点数，直接删剩余的
        // if (patched >= toBePatched) { 
        //     unmount(prevChild, parentComponent, parentSuspense, //true)
        //        continue
        //     }
                const prevChild = c1[i]
                let newIndex

            1. 通过key匹配新旧节点，newIndex就是标识节点在c2中的位置 
                此时上面得到的keyToNewIndexMap开始发挥作用
                if (prevChild.key != null) {
                newIndex = keyToNewIndexMap.get(prevChild.key)
                } else { 

            3. 如果,老节点的key不存在，遍历剩下的所有新节点。什么情况下老节点的key会不存在呢？？
                for (j = s2; j <= e2; j++) {
                // newIndexToOldIndexMap里面本身初始化的都是0
            // 这里=== 0  代表 新节点没有被patch
                    if (  newIndexToOldIndexMap[j - s2] === 0 && 
                    isSameVNodeType(prevChild, c2[j] as VNode)
                    ) { 
                3.1  找到与当前老节点对应的新节点，记录newIndex 
                    newIndex = j
                    break
                    }
                }
                }

        2.1 newIndex 无值，代表没有找到与老节点对应的新节点，删除节点
        //拓展：void 0是undefined备选方案。字节更少
        // void 后面随便跟上一个表达式，返回的都是 undefined
                if (newIndex === void 0) { 
                unmount()
                } else {

        2.2 注意：i + 1,0 有特殊用途。把老节点的索引，记录在存放新节点的数组中
                newIndexToOldIndexMap[newIndex - s2] = i + 1

        2.3 看这里：记录maxNewIndexSoFar
            如果某节点的索引大于之前节点的索引
            代表有元素移动了moved = true
                if (newIndex >= maxNewIndexSoFar) {
                    maxNewIndexSoFar = newIndex
                } else {
                    moved = true
                }

        2.4 找到新的节点进行patch节点 ，
        建立对应关系，执行patch将旧dom上的el（真实dom的引用）和其他给到新节点
                patch(prevChild)
                patched++
                }
        }
        // 1. 
        // 遍历c1待处理的节点, 在c2中有没有相同key的节点
        // 没有就直接删掉unmount，有就记录改节点的索引
        // 在找的过程中发现旧节点没有key？？那就嵌套遍历新节点c2找到对应关系
        // 2. 
        // 把旧节点在c1的位置（非索引，因为0要代表新增节点），存放在newIndexToOldIndexMap
    ```
    最后一步遍历c2，该移动的移动该新增的新增
    ```js
        const increasingNewIndexSequence = moved
                ? getSequence(newIndexToOldIndexMap)
                : EMPTY_ARR
            j = increasingNewIndexSequence.length - 1
        注意：这里是从后往前遍历
        i = 3  s2 = 2
        nextIndex = 5
        anchor 是找到h节点的后一个，也就是f节点插入。小于新节点长度实际上就是移动到最后
        for (i = toBePatched - 1; i >= 0; i--) {
            const nextIndex = s2 + i
            const nextChild = c2[nextIndex]
            const anchor =
                nextIndex + 1 < c2.length ? (c2[nextIndex + 1] as VNode).el : parentAnchor

        1. 这里 === 0 表示 c2 中的节点在 c1 中没有对应的节点，就新增它
            if (newIndexToOldIndexMap[i] === 0) {
                patch(null)
            } else if (moved) {
        2. 移动， 怎么最小化移动呢？最长递增子序列对应的元素不需要移动
                // j < 0  --> 最长递增子序列为 [] 
                if (j < 0 || i !== increasingNewIndexSequence[j]) {
                    move(nextChild, container, anchor)
                } else {
                    j--
                }
            }
        }
    ```
     
- nextTick
  > 其实原理都大致相同，3.x的版本中改了一下写法，添加了维护队列的方法。 源码地址：packages/runtime-core/src/scheduler.ts；执行顺序：`queueJob` -> `queueFlush` -> `flushJobs` -> `nextTick参数的 fn`
  - 3.x 
    ```js
        //在数据进行更新的时候， 会调用下面这个方法；地址 packages/runtime-core/src/renderer.ts - 1400多行， 我修改后乱行了
        function baseCreateRenderer(){
            const setupRenderEffect: SetupRenderEffectFn = (...) => {
                const effect = new ReactiveEffect(
                componentUpdateFn,
                () => queueJob(instance.update), // 当作参数传入
                instance.scope
                )
            }
        }

        // packages/reactivity/src/effect.ts - 330行
        export function triggerEffects(
            ...
            if (effect.scheduler) {
                effect.scheduler()
            } else {
                effect.run()
            }
        }

        // queueJob()
        //该方法负责维护主任务队列，接受一个函数作为参数，为待入队任务，会将参数 push 到 queue 队列中，有唯一性判断。会在当前宏任务执行结束后，清空队列
        const queue: SchedulerJob[] = []

        export function queueJob(job: SchedulerJob) {
        // 主任务队列为空 或者 有正在执行的任务且没有在主任务队列中  && job 不能和当前正在执行任务及后面待执行任务相同
        if ((!queue.length ||
            !queue.includes( job, isFlushing && job.allowRecurse ? flushIndex + 1 : flushIndex )
            ) && job !== currentPreFlushParentJob
        ) {
            // 可以入队就添加到主任务队列
            if (job.id == null) {
            queue.push(job)
            } else {
            // 否则就删除
            queue.splice(findInsertionIndex(job.id), 0, job)
            }
            // 创建微任务
            queueFlush()
        }
        }

        // queueFlush()
        // 该方法负责尝试创建微任务，等待任务队列执行
        let isFlushing = false // 是否正在执行
        let isFlushPending = false // 是否正在等待执行
        const resolvedPromise: Promise<any> = Promise.resolve() // 微任务创建器
        let currentFlushPromise: Promise<void> | null = null // 当前任务

        function queueFlush() {
        // 当前没有微任务
        if (!isFlushing && !isFlushPending) {
            // 避免在事件循环周期内多次创建新的微任务
            isFlushPending = true
            // 创建微任务，把 flushJobs 推入任务队列等待执行
            currentFlushPromise = resolvedPromise.then(flushJobs)
        }
        }

        // flushJobs()
        // 该方法负责处理队列任务，主要逻辑如下：

        // 1. 先处理前置任务队列
        // 2. 根据 Id 排队队列
        // 3. 遍历执行队列任务
        // 4. 执行完毕后清空并重置队列
        // 5. 执行后置队列任务
        // 6. 如果还有就递归继续执行
        function flushJobs(seen?: CountMap) {
        isFlushPending = false // 是否正在等待执行
        isFlushing = true // 正在执行
        if (__DEV__) seen = seen || new Map() // 开发环境下
        flushPreFlushCbs(seen) // 执行前置任务队列
        // 根据 id 排序队列，以确保
        // 1. 从父到子，因为父级总是在子级前面先创建
        // 2. 如果父组件更新期间卸载了组件，就可以跳过
        queue.sort((a, b) => getId(a) - getId(b))
        try {
            // 遍历主任务队列，批量执行更新任务
            for (flushIndex = 0; flushIndex < queue.length; flushIndex++) {
            const job = queue[flushIndex]
            if (job && job.active !== false) {
                if (__DEV__ && checkRecursiveUpdates(seen!, job)) {
                continue
                }
                callWithErrorHandling(job, null, ErrorCodes.SCHEDULER)
            }
            }
        } finally {
            flushIndex = 0 // 队列任务执行完，重置队列索引
            queue.length = 0 // 清空队列
            flushPostFlushCbs(seen) // 执行后置队列任务
            isFlushing = false  // 重置队列执行状态
            currentFlushPromise = null // 重置当前微任务为 Null
            // 如果主任务队列、前置和后置任务队列还有没被清空，就继续递归执行
            if ( queue.length || pendingPreFlushCbs.length || pendingPostFlushCbs.length ) {
            flushJobs(seen)
            }
        }
        }
        // flushPreFlushCbs()  该方法负责执行前置任务队列，说明都写在注释里了

        export function flushPreFlushCbs( seen?: CountMap, parentJob: SchedulerJob | null = null) {
        // 如果待处理的队列不为空
        if (pendingPreFlushCbs.length) {
            currentPreFlushParentJob = parentJob
            // 保存队列中去重后的任务为当前活动的队列
            activePreFlushCbs = [...new Set(pendingPreFlushCbs)]
            // 清空队列
            pendingPreFlushCbs.length = 0
            // 开发环境下
            if (__DEV__) { seen = seen || new Map() }
            // 遍历执行队列里的任务
            for ( preFlushIndex = 0; preFlushIndex < activePreFlushCbs.length; preFlushIndex+ ) {
            // 开发环境下
            if ( __DEV__ && checkRecursiveUpdates(seen!, activePreFlushCbs[preFlushIndex])) {
                continue
            }
            activePreFlushCbs[preFlushIndex]()
            }
            // 清空当前活动的任务队列
            activePreFlushCbs = null
            preFlushIndex = 0
            currentPreFlushParentJob = null
            // 递归执行，直到清空前置任务队列，再往下执行异步更新队列任务
            flushPreFlushCbs(seen, parentJob)
        }
        }

        // flushPostFlushCbs()

        let activePostFlushCbs: SchedulerJob[] | null = null

        export function flushPostFlushCbs(seen?: CountMap) {
        // 如果待处理的队列不为空
        if (pendingPostFlushCbs.length) {
            // 保存队列中去重后的任务
            const deduped = [...new Set(pendingPostFlushCbs)]
            // 清空队列
            pendingPostFlushCbs.length = 0
            // 如果当前已经有活动的队列，就添加到执行队列的末尾，并返回
            if (activePostFlushCbs) {
            activePostFlushCbs.push(...deduped)
            return
            }
            // 赋值为当前活动队列
            activePostFlushCbs = deduped
            // 开发环境下
            if (__DEV__) seen = seen || new Map()
            // 排队队列
            activePostFlushCbs.sort((a, b) => getId(a) - getId(b))
            // 遍历执行队列里的任务
            for ( postFlushIndex = 0; postFlushIndex < activePostFlushCbs.length; postFlushIndex++ ) {
            if ( __DEV__ && checkRecursiveUpdates(seen!, activePostFlushCbs[postFlushIndex])) {
                continue
            }
            activePostFlushCbs[postFlushIndex]()
            }
            // 清空当前活动的任务队列
            activePostFlushCbs = null
            postFlushIndex = 0
        }
        }

    ```


  我们使用的Vue.nextTick,this.$nextTick都是nextTick一样的，但是在vue3里面，可能this要通过getCurrentInstance去过去ctx, 然后调用ctx.$nextTick, 其实整体的流程也还算简单，vue内部有一个事件队列，初始化、视图更新的时候就放进队列一些时间，按顺序执行，只是把nextTick的回掉函数放到了视图更新后的一个事件里面， 然后再继续执行下面的操作， 可以看下queuePostRenderEffect这个注册的事件队列。

- reactive
- effect
- 加油服役typescript面试题
- react fiber
- Vue3 和 Vue2 响应式区别
  - 使用上的区别
  - 源码目录结构，原理的区别
    - vue2的Object.defineProperty，并针对数组的响应式重写
    - vue3的proxy一并代理了数组、对象。
  - 依赖收集的区别
    - vue2通过Dep、Watcher、Observe来进行收集、派发、监听
    - vue3通过track进行收集，trigger进行派发，proxy的监听机制
  - 缺点区别
    - vue2针对数组的重写
    - vue3的es6导致不兼容其他浏览器（weakmap、map、set、weakset）
- vue为啥不要用index作key
  本身key的出现是提高虚拟dom的对比减少其他函数的执行(更高效的更新虚拟dom，在精准的程度上，patch会减少操作)，比如updateAttrs，updateClass,updateDOMProps,updateDOMListeners,updateStyle,updateDirectives。使用索引导致vue没有办法区分它，Vue 判断两个节点是否相同时主要判断两者的元素类型和 key 等，如果不设置 key，就可能永远认为这两个是相同节点，只能去做更新操作，就造成大量不必要的 DOM 更新操作，明显是不可取的
- 虚拟dom到底啥嘎哈的?
    - 用JS对象模拟DOM（虚拟DOM）
    - 把此虚拟DOM转成真实DOM并插入页面中（render）
    - 如果有事件发生修改了虚拟DOM，比较两棵虚拟DOM树的差异，得到差异对象（diff）
    - 把差异对象应用到真正的DOM树上（patch）
    - diff算法的目的是为了减少 DOM 操作的性能开销，我们要尽可能的复用 DOM 元素。所以我们需要判断出是否有节点需要移动，应该如何移动以及找出那些需要被添加或删除的节点。