### vue中nextTick的原理

- 作用
    在dom更新之后会执行一个回调，拿到最新的数据

- 原理猜测
    使用了h5中MutationObserver这个API，可以监听到dom元素的改动

- 源码
    ```js
        // src/core/util/next-tick.js中59行
        // Use MutationObserver where native Promise is not available,
        // e.g. PhantomJS, iOS7, Android 4.4
        // (#6466 MutationObserver is unreliable in IE11)
        let counter = 1
        const observer = new MutationObserver(flushCallbacks)
        const textNode = document.createTextNode(String(counter))
        observer.observe(textNode, {
            characterData: true
        })
        timerFunc = () => {
            counter = (counter + 1) % 2
            textNode.data = String(counter)
        }
        isUsingMicroTask = true
    ```
    在可以使用MutationObserver的时候，会创建一个MutationObserver实例，然后创建一个textNode，然后监听这个文本节点，这个文本节点改动之后，执行回调

- 问题
    为什么监听到textNode改变之后，就可以当作dom更新并执行回调，这个是什么依据呢？这个和事件循环以及vue的异步批量更新渲染有一定的联系

- 原因
    - 我们知道MutationObserver属于微队列。如果把在每个组件的渲染执行当成一次事件循环，界面的整体更新渲染在世界循环结束之后执行。
    - 在异步批量更新渲染会使用promise把所有更新压入微队列，这个时候由于MutationObserver也是一个微队列，会把他压入微队列的队尾，这样在所有的render以及diff更新真实dom结束之后，nextTick会执行，立刻拿到最新数据以及最新dom。
    - 这个时候监听到dom改变即是当前组件中所有dom元素全部执行的时刻，可以放心执行调用回调函数。

- 还有一个问题，hhhhh
    探究清楚之后，我们稍微多想一点就会发现一个问题。刚才的解决策略的核心是微队列，MutationObserver说到底也只是利用了他微队列的特性，其实在队尾执行就知道所有dom更新结束咯，那么检测他dom更改的意义在哪？其实由于MutationObserver是H5新增的特性，在ios的开发中有bug，所以vue在2.5版本中已经把MutationObserver相关的代码干掉了，使用promise解决这个问题。但是由于promise是ES6的新特性，存在一定限制，所以vue中自然而然存在一些优雅的降级策略，在所有微队列都不支持的时候，用宏队列的setTimeout来兜底。

- 结论
    绕了一圈，最后发现nextTick实现的原理其实是和vue异步批量渲染中微队列策略息息相关，在一次事件循环中在微队列的最后执行，确保dom更新后执行回调函数。所以我们的结论 ko no microtask 哒

