---
title: Vue 解析
categories: FE
author: 🐑
tags:
  - vue
  - js
---
# Vue 解析

## 初始化

```js
// /core/instance/index.js
// 构造函数 入口
function Vue (options)
```

### Vue.prototype 属性, 方法

#### <h4 id="_init">_init()</h4>

/core/instance/init.js

```js
function (options) {
    const vm = this
    // 实例唯一标识
    vm._uid = uid++
    // Vue 实例标记
    vm._isVue = true
    // 判断是否为内部模板
    if (options && options._isComponent) {
      // 初始化内部模板
      initInternalComponent(vm, options)
    } else {
      // 合并 构造器 options 到 参数 options
      vm.$options = mergeOptions(
        // 解析 构造器 options 
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    vm._self = vm
    // 初始化生命周期
    initLifecycle(vm)
    // 初始化事件
    initEvents(vm)
    // 初始化render
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm)
    initState(vm)
    initProvide(vm) 
    callHook(vm, 'created')
    
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
}
```

**相关**

- [_uid]($_uid)
- [_isVue](#_isVue)
- [_isComponent](#_isComponent)
- [$options](#$options)
- [_self](#_self)
- [$mount](#$mount)

##### initInternalComponent()

```js
function initInternalComponent (vm: Component, options: InternalComponentOptions)
```

##### resolveConstructorOptions()

 ```js
// 解析 构造器 options
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
 ```



**相关**

- [options](#options)
- [super](#super)

##### mergeOptions()

```js
// 合并 option
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  // 支持 构造函数构建新的实例 主要用于 下文 extend 和 mixins 的 递归
  // 实例: https://jsfiddle.net/Dared/y7hunx3u/
  if (typeof child === 'function') {
    child = child.options
  }
  // 统一 props 的格式
  normalizeProps(child, vm)
  // 统一 inject 的格式
  normalizeInject(child, vm)
  // 统一 directives 的格式
  normalizeDirectives(child)
  const extendsFrom = child.extends
  if (extendsFrom) {
    // 有申明扩展的组件 合并扩展的 options 到父 options
    parent = mergeOptions(parent, extendsFrom, vm)
  }
  if (child.mixins) {
    // 有申明混入的组件 合并混入的 options 到父 options
    for (let i = 0, l = child.mixins.length; i < l; i++) {
      parent = mergeOptions(parent, child.mixins[i], vm)
    }
  }
  const options = {}
  let key
  for (key in parent) {
    // 循环添加 父 options 到 options
    mergeField(key)
  }
  for (key in child) {
    // 上面循环过的 key 不需要再次添加
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    // 使用 Vue.config.optionMergeStrategie[key] 的策略合并
    // 参考 Vue.config
    // defaultStrat 默认策略: 当 child 存在时 使用 child 替换 parent , 否则使用 parent
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  // 返回 合并后 options
  return options
}
```

**相关** 

- [extends](#extends)
- [mixins](#mixins)
- [config](#config)

###### normalizeProps()

**统一 props 的格式**

props 支持的格式

```js
// 简单语法
{
  props: ['size', 'myMessage']
}

// 对象语法，提供校验
{
  props: {
    // 检测类型
    height: Number,
    // 检测类型 + 其他验证
    age: {
      type: Number,
      default: 0,
      required: true,
      validator: function (value) {
        return value >= 0
      }
    }
  }
}
```

转换统一格式

```js
// 统一 props 的格式
function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  // 针对数组类型的转换
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      // 数组类型要求值必须是 string
      if (typeof val === 'string') {
        name = camelize(val)
        // 将数组类型转换为标准对象
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    // 对象类型处理
    for (const key in props) {
      val = props[key]
      // 将对象的 key 转驼峰
      name = camelize(key)
      // 非标准对象改成标准对象
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    // 拒绝非[]和{}类型
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  // 替换原来的 props
  options.props = res
}
```

**相关** 

- [props](#props)

###### normalizeInject()

**统一 inject 的格式**

inject 支持格式

```js
{
  inject: ['foo']
}
{
  inject: {
    foo: 'foo',
    foo: { 
      default: 'bar',
      form: 'foo',
      default: () => [1, 2, 3]
    }
  }
}
```

转换统一格式

```js
// 统一 inject 的格式
function normalizeInject (options: Object, vm: ?Component) {
  // 格式前 inject
  const inject = options.inject
  if (!inject) return
  // 格式后 inject 存入 options.inject
  const normalized = options.inject = {}
  if (Array.isArray(inject)) {
    for (let i = 0; i < inject.length; i++) {
      // 数据类型转换为对象
      normalized[inject[i]] = { from: inject[i] }
    }
  } else if (isPlainObject(inject)) {
    // 对象类型格式化
    for (const key in inject) {
      const val = inject[key]
      normalized[key] = isPlainObject(val)
        // 标准对象加上 form
        ? extend({ from: key }, val)
        // 非标准对象转换
        : { from: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "inject": expected an Array or an Object, ` +
      `but got ${toRawType(inject)}.`,
      vm
    )
  }
}
```

**相关** 

- [inject](#inject)

###### normalizeDirectives()

**统一 directives 的格式**

directives 支持格式

```js
{
  directives: function () {}
}
{
  directives: {
    bind: function () {},
    inserted: function () {},
    update: function () {},
    componentUpdated: function () {},
    unbind: function () {}
  }
}
```

转换统一格式

```js
// 统一 directives 的格式
function normalizeDirectives (options: Object) {
  const dirs = options.directives
  if (dirs) {
    for (const key in dirs) {
      const def = dirs[key]
      // 把 function 类型转换为 {bind, update} 的标注形式
      if (typeof def === 'function') {
        dirs[key] = { bind: def, update: def }
      }
    }
  }
}
```

**相关** 

- [directives](#directives)

##### initLifecycle()

```js
export function initLifecycle (vm: Component) {
  const options = vm.$options

  // locate first non-abstract parent
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }
  // 保存父节点
  vm.$parent = parent
  // 保存根节点
  vm.$root = parent ? parent.$root : vm
  // 初始化子节点
  vm.$children = []
  // 初始化 ref 描述节点
  vm.$refs = {}
  // 初始化 内部对象
  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

**相关** 

- [parent](#parent)
- [abstract](#abstract)

#### <h4 id="$data">$data</h4>   `readonly` 

/core/instance/state.js

指向 实例 _data

#### <h4 id="$prop">$prop</h4> `readonly`

/core/instance/state.js

指向 实例 _props

#### <h4 id="$set">$set()</h4>

/core/instance/state.js

#### <h4 id="$delete">$delete()</h4>

/core/instance/state.js

#### <h4 id="$watch">$watch()</h4>

/core/instance/state.js

#### <h4 id="$on">$on()</h4>

/core/instance/events.js

#### <h4 id="$once">$once()</h4>

/core/instance/events.js

#### <h4 id="$off">$off()</h4>

/core/instance/events.js

#### <h4 id="$emit">$emit()</h4>

/core/instance/events.js

#### <h4 id="_update">_update()</h4>

/core/instance/lifecycle.js

#### <h4 id="$forceUpdate">$forceUpdate()</h4>

/core/instance/lifecycle.js

#### <h4 id="$destroy">$destroy()</h4>

/core/instance/lifecycle.js

##### _o()

/core/instance/render-helpers/index.js

##### _n()

/core/instance/render-helpers/index.js

##### _s()

/core/instance/render-helpers/index.js

##### _l()

/core/instance/render-helpers/index.js

##### _t()

/core/instance/render-helpers/index.js

##### _q()

/core/instance/render-helpers/index.js

##### _i()

/core/instance/render-helpers/index.js

##### _m()

/core/instance/render-helpers/index.js

##### _f()

/core/instance/render-helpers/index.js

##### _k()

/core/instance/render-helpers/index.js

##### _b()

/core/instance/render-helpers/index.js

##### _v()

/core/instance/render-helpers/index.js

##### _e()

/core/instance/render-helpers/index.js

##### _u()

/core/instance/render-helpers/index.js

##### _g()

/core/instance/render-helpers/index.js

#### <h4 id="$nextTick">$nextTick()</h4>

/core/instance/render.js

#### <h4 id="_render">_render()</h4>

/core/instance/render.js

#### <h4 id="$isServer">$isServer</h4>  `readonly`

/core/index.js

#### <h4 id="$ssrContext">$ssrContext</h4>  `readonly`

/core/index.js

#### <h4 id="FunctionalRenderContext">FunctionalRenderContext()</h4>

/core/index.js

#### <h4 id="$mount">$mount</h4>

/platforms/runtime/index.js

#### Vue 属性, 方法

#### <h4 id="config">config</h4> `readonly`

/core/global-api/index.js

- optionMergeStrategies

  options 合并策略, 没有添加的采用 child || parent 的策略 

  参考 

  /core/util/options.js 提供了一下默认合并策略

  - data

    > 进行浅合并 (一层属性深度)，在和组件的数据发生冲突时以组件数据优先。

    function 执行后合并, 这里的 `一层属性深度` 在 to 和 from 都为对象的时候会继续递归

    `一层属性深度` 不是 `一层深度`

  - beforeCreate

  - created

  - beforeMount

  - mounted

  - beforeUpdate

  - updated

  - beforeDestroy

  - destroyed

  - activated

  - deactivated

  - errorCaptured

    > 同名钩子函数将混合为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子**之前**调用。

    将方法转数字 concat 到 parent 的 钩子数组里面

  - components

  - directives

  - filter

    以 parent 为原型, 混入 child 属性

    ```js
    return extend(Object.create(parent), child)
    ```

  - watch

    同 key 监听合并成数组

  - props

  - methods

  - inject

  - computed

     child 覆盖 parent 策略

  - provide

    和 data 相同的 合并策略

- silent

- productionTip

- devtools

- performance

- errorHandler

- warnHandler

- ignoredElements

- keyCodes

- isReservedTag

- isReservedAttr

- isUnknownElement

- getTagNamespace

- parsePlatformTagName

- mustUseProp

- _lifecycleHooks

#### <h4 id="util">util</h4>

/core/global-api/index.js

#### <h4 id="set">set()</h4>

/core/global-api/index.js

#### <h4 id="delete">delete()</h4>

/core/global-api/index.js

#### <h4 id="nextTick">nextTick()</h4>

/core/global-api/index.js

#### <h4 id="options">options</h4>

/core/global-api/index.js

- components

  - KeepAlive
  - Transition
  - TransitionGroup

- directives

  - model
  - show

- filters

- _base

  ```js
  Vue.options._base = Vue
  ```

#### <h4 id="use">use()</h4>

/core/global-api/use.js

#### <h4 id="mixin">mixin()</h4>

/core/global-api/mixin.js

#### <h4 id="cid">cid</h4>

/core/global-api/extend.js

#### <h4 id="extend">extend()</h4>

/core/global-api/extend.js

#### <h4 id="component">component()</h4>

/core/global-api/assets.js

#### <h4 id="directive">directive()</h4>

/core/global-api/assets.js

#### <h4 id="filter">filter()</h4>

/core/global-api/assets.js

## 创建

```Js
function Vue (options) {
  this._init(options)
}
```

**相关**

- [_init()](#_init) 

### 构建参数 `options`

#### <h4 id="_isComponent">_isComponent<h5>

标识内部模板

**相关**

- [_init()](#_init) 

#### <h4 id="props">props</h4> `Array<string> | Object`

> props 可以是数组或对象，用于接收来自父组件的数据。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选项，如类型检测、自定义校验和设置默认值。

**相关**

- [_init()](#_init) 

#### <h4 id="inject">inject</h4>

> 这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

**相关**

- [_init()](#_init) 

#### <h4 id="directives">directives</h4>

> 参考文档 自定义指令
>
> https://cn.vuejs.org/v2/guide/custom-directive.html

**相关**

- [_init()](#_init) 

#### <h4 id="extends">extends</h4>

> 允许声明扩展另一个组件(可以是一个简单的选项对象或构造函数)，而无需使用 `Vue.extend`。这主要是为了便于扩展单文件组件。

**相关**

- [_init()](#_init) 

#### <h4 id="mixins">mixins</h4>

> 混入 (mixins) 是一种分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。

**相关**

- [_init()](#_init) 

#### <h4 id="el">el</h4>

**相关**

- [_init()](#_init) 

#### <h4 id="parent">parent</h4>

**相关**

- [_init()](#_init) 

#### <h4 id="abstract">abstract</h4>

#### _parentListeners

#### _parentVnode

**相关**

- [_init()](#_init) 

### 构建实例 `new Vue()`

#### <h4 id="_uid">_uid</h4>

每个vue实例有一个唯一的uid

/core/instance/init.js

#### <h4 id="_isVue">_isVue</h4>

Vue 实例标记

/core/instance/init.js

#### <h4 id="$options">$options</h4>

合并 继承, 混入,构建参数 等 之后的 options

```
_propKeys
```

**相关**

- [_init()](#_init) 

#### <h4 id="_self">_self</h4>

**相关**

- [_init()](#_init) 

#### <h4 id="super">super</h4>

父节点

#### $root

根节点

#### $children

子节点 数组

#### $refs

ref 标记 节点 

#### _watcher

#### _inactive

#### _directInactive

#### _isMounted

是否 渲染 到 dom

#### _isDestroyed

是否 销毁

#### _isBeingDestroyed

是否 被 销毁

#### _events

#### _hasHookEvent

#### _vnode

#### _staticTrees

#### _c

#### $createElement

#### $slots

#### $scopedSlots

#### _provided

#### _watchers

#### _props

```
_props
```