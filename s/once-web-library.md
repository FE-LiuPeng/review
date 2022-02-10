### <center>一更新文--前端框架库</center>

#### vue2/3，react diff对比
- react-diff
  1. 实现原理
    > React的思路是递增法。通过对比新的列表中的节点，在原本的列表中的位置是否是递增，来判断当前节点是否需要移动。



- nextTick
  > 其实原理都大致相同，3.x的版本中改了一下写法，添加了维护队列的方法。 源码地址：packages/runtime-core/src/scheduler.ts；执行顺序：`queueJob` -> `queueFlush` -> `flushJobs` -> `nextTick参数的 fn`
  - 2.x
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
    ```
- diff
- reactive
- effect
- nextTick
- 加油typescript面试题


用JS对象模拟DOM（虚拟DOM）
把此虚拟DOM转成真实DOM并插入页面中（render）
如果有事件发生修改了虚拟DOM，比较两棵虚拟DOM树的差异，得到差异对象（diff）
把差异对象应用到真正的DOM树上（patch）