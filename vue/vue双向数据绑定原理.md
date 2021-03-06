### vue双向数据绑定原理

- 什么是双向绑定数据
    view和model有一方改变的时候都会通知另外一个进行更新。也就是数据变化更新视图，视图变化也会更新数据。

- 设计思想
> 观察者模式

- data -> model
> data改变model主要是vue内部实现了一个数据响应化，下面从源码分析一下数据响应式

    ```js
        // vue/src/core/observer/index.js
        // 既然说到响应式，就去响应式的目录中寻找答案

        /**
        * Attempt to create an observer instance for a value,
        * returns the new observer if successfully observed,
        * or the existing observer if the value already has one.
        */
        export function observe (value: any, asRootData: ?boolean): Observer | void {
        if (!isObject(value) || value instanceof VNode) {
            return
        }
        let ob: Observer | void
        if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
            ob = value.__ob__
        } else if (
            shouldObserve &&
            !isServerRendering() &&
            (Array.isArray(value) || isPlainObject(value)) &&
            Object.isExtensible(value) &&
            !value._isVue
        ) {
            ob = new Observer(value)
        }
        if (asRootData && ob) {
            ob.vmCount++
        }
        return ob
        }

        // 在数据初始化initState的时候会调用observe方法将data注入
        // 这里会创建观察者实例并把data再次注入

        /**
        * Observer class that is attached to each observed
        * object. Once attached, the observer converts the target
        * object's property keys into getter/setters that
        * collect dependencies and dispatch updates.
        */
        export class Observer {
            value: any;
            dep: Dep;
            vmCount: number; // number of vms that have this object as root $data

            constructor (value: any) {
                this.value = value
                this.dep = new Dep()
                this.vmCount = 0
                def(value, '__ob__', this)
                if (Array.isArray(value)) {
                if (hasProto) {
                    protoAugment(value, arrayMethods)
                } else {
                    copyAugment(value, arrayMethods, arrayKeys)
                }
                    this.observeArray(value)
                } else {
                    this.walk(value)
                }
            }

            /**
            * Walk through all properties and convert them into
            * getter/setters. This method should only be called when
            * value type is Object.
            */
            walk (obj: Object) {
                const keys = Object.keys(obj)
                for (let i = 0; i < keys.length; i++) {
                    defineReactive(obj, keys[i])
                }
            }

            /**
            * Observe a list of Array items.
            */
            observeArray (items: Array<any>) {
                for (let i = 0, l = items.length; i < l; i++) {
                    observe(items[i])
                }
            }
        }

        // 这里会根据data是数组还是对象执行不同的方法，当data是对象的时候会执行defineReactive方法绑定响应式

        /**
        * Define a reactive property on an Object.
        */
        export function defineReactive (
            obj: Object,
            key: string,
            val: any,
            customSetter?: ?Function,
            shallow?: boolean
        ) {
            const dep = new Dep()

            const property = Object.getOwnPropertyDescriptor(obj, key)
            if (property && property.configurable === false) {
                return
            }

            // cater for pre-defined getter/setters
            const getter = property && property.get
            const setter = property && property.set
            if ((!getter || setter) && arguments.length === 2) {
                val = obj[key]
            }

            let childOb = !shallow && observe(val)
            Object.defineProperty(obj, key, {
                enumerable: true,
                configurable: true,
                get: function reactiveGetter () {
                const value = getter ? getter.call(obj) : val
                if (Dep.target) {
                    dep.depend()
                    if (childOb) {
                    childOb.dep.depend()
                    if (Array.isArray(value)) {
                        dependArray(value)
                    }
                    }
                }
                return value
                },
                set: function reactiveSetter (newVal) {
                const value = getter ? getter.call(obj) : val
                /* eslint-disable no-self-compare */
                if (newVal === value || (newVal !== newVal && value !== value)) {
                    return
                }
                /* eslint-enable no-self-compare */
                if (process.env.NODE_ENV !== 'production' && customSetter) {
                    customSetter()
                }
                // #7981: for accessor properties without setter
                if (getter && !setter) return
                if (setter) {
                    setter.call(obj, newVal)
                } else {
                    val = newVal
                }
                childOb = !shallow && observe(newVal)
                dep.notify()
                }
            })
        }

        // 这里使用了defineProperty来实现响应式，get负责监听数据以及手机依赖，set用来响应改变以及通知view更新

    ```

- view -> data
> vue中在每个组件实现了一个watcher实例，用来更新，当如input的value之类的视图发生变化，vue的compiler会根据监听的事件通知watcher，进行更新