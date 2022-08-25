---
title: 在Vue3中实现React Hooks
date: 2022-08-25 13:35:23  
categories:  Vue3
tags:
- Vue3
- React
- Hooks
---

## 前言
- 最近在逛掘金的时候看到些比较有意思的文章可以在Vue3中调用React Hooks，所以就学习了一下发现了设计的巧妙之处，虽然我们可能永远用不到这个，但是学习一下实现的原理还是挺不错的。
- 首先本文不会过度深入讲解只属于 React 或者只属于 Vue 的内容，所以只懂 React 或者只懂 Vue 的同学都可以畅通无阻地阅读本文。  


## 在 Vue3 函数式组件中的 React-style Hooks
我们先来看一段代码  
```
import { useState, useReducer, useEffect, useLayoutEffect } from "vue-hooks-api";

const FunctionalComponent = (props, context) => {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useReducer((x) => x + 1, 1);
  const [count3, setCount3] = useReducer((x) => x + 1, 2);

  useEffect(() => {
    console.log("useEffect", count2);
  }, [count2]);

  useLayoutEffect(() => {
    console.log("useLayoutEffect", count2);
  }, [count2]);

  return (
    <>
      <button onClick={() => setCount1(2)} {...props}>
        count1:{count1}
      </button>
      <button onClick={() => setCount2()} {...props}>
        count2:{count2}
      </button>
      <button onClick={() => setCount3()} {...props}>
        count3:{count3}
      </button>
    </>
  );
};

export default FunctionalComponent;
```
React 的同学可能以为这是一个 React 的函数组件，其实不是，这是一个 Vue3 的函数式组件，通过 vue-hooks-api 包提供的 useState, useReducer, useEffect, useLayoutEffect 的 Hooks 函数，就可以在 Vue3 的函数式组件中使用了，再通过 JSX 方式使用则看起来基本可以跟 React Hooks 一样了。  

关于 vue-hooks-api npm 包  
vue-hooks-api npm 包是本文作者发布的一个 React 风格的 Vue3 Hooks 包，目前只可使用于 Vue3 函数式组件，跟 React 的函数式组件的 Hooks 使用方式一致。  
可以通过 yarn 方式安装体验。  
```
yarn add vue-hooks-api
```
注意，此 npm 包目前只是一个实验性产品，旨在探讨如何在 Vue3 的函数组件中实现 React 式的函数组件 Hooks，请慎用于生产环境。 

下文也将围绕这个 vue-hooks-api npm 包是如何实现的进行讲解。  

## React Hooks 的本质
首先 React Hooks 只可以使用在 React 的函数组件上，在 React Hooks 出现之前 React 的函数组件是不可以存储属于自己的数据状态的，因故也不可以进行数据逻辑的复用。直到 React Hooks 的出现，在 React 的函数组件上就可以进行存储属于它自己的数据状态了，进而可以达到数据逻辑的复用。这也是 React Hooks 的作用，可以进行数据逻辑的复用。  

那么为什么 React 可以做到在函数组件上进行存储数据状态的呢？首先 React 函数组件的本质是一个函数，React 函数的更新就是重新执行 React 函数组件得到新的虚拟 DOM 数据。那么要在 React 函数组件上存储属于这个函数组件自己的数据，本质就是在一个函数上存储属于这个函数的数据，在这个函数的后续执行的时候还可以获取到它自己内部的变量数据，并且不会和其他函数组件的内部的变量数据发生冲突，这其中最好的实现方式就是实现一个闭包函数。  

React Hooks 的最简实现
```
// Hooks
function useReducer(reducer, initalState) {
    let hook = initalState
    const dispatch = (action) => {
        hook = reducer(hook, action)
        // 关键，执行 setCount 函数的时候会重新执行 FunctionComponent 函数
        FunctionComponent()
    }
    return [hook, dispatch]
}
// 函数组件
function FunctionComponent() {
   const [count, setCount] = useReducer(x => x + 1, 0)
    
   return {count, setCount}
}

const result = FunctionComponent()
// 执行 setCount 会从新执行 FunctionComponent
result.setCount()
```
通过上面 React Hooks 的最简模型可以知道执行组件函数 FunctionComponent 可以看成从 Hooks 返回了两个变量 count 和 setCount，count 很明显是拿来展示使用的，setCount 则是拿来给用户交互使用的，当用户执行 setCount 的时候 FunctionComponent 会重新执行。其中关键的就是hi执行 setCount 函数的时候会重新执行 FunctionComponent 函数。  

上述 React Hooks 的最简模型还存在一个问题，当用户执行 setCount 的时候 FunctionComponent 重新执行的时候，hook 会被一直初始化，值不能进行迭代。那么我们知道 React 当中一个函数组件就是一个 Fiber 节点，所以可以把 hook 存储在 Fiber 节点上。因为 Fiber 节点变量相对于这个 Hook 函数来说，就是一个全局变量。  
```
// Fiber 节点
const Fiber = {
    type: FunctionComponent, // Fiber 节点上的 type 属性是组件函数
    memorizedState: null // Fiber 节点上的 memorizedState 属性是 Hooks
}
// Hooks
function useReducer(reducer, initalState) {
    // 初始化的时候，如果 Fiber 节点的 Hooks 不存在则进行设置
    if(!Fiber.memorizedState) Fiber.memorizedState = initalState
    const dispatch = (action) => {
        Fiber.memorizedState = reducer(Fiber.memorizedState, action)
        // 关键，执行 setCount 函数的时候会重新执行 FunctionComponent 函数
        Fiber.type()
    }
    return [Fiber.memorizedState, dispatch]
}
// 函数组件
function FunctionComponent() {
    const [count, setCount] = useReducer(x => x + 1, 0)
    console.log("渲染的count:", count) 
    return {count, setCount}
}

const result = Fiber.type() // 打印 0
// 执行 setCount 会从新执行 FunctionComponent
result.setCount() // 打印 1
result.setCount() // 打印 2
result.setCount() // 打印 3

```

经过上述代码修改之后，一个最简单的 React Hooks 的模型就实现完成了，这也是 React Hooks 的本质。值得注意的是其中 reducer 的实现跟 Redux 的 reducer 的实现是很相似的，这是因为它们是同一个作者开发的功能的缘故。  

## 问题
实际上React Hooks的设计远比这个复杂多了，上述代码还存在两个比较大的问题，一个问题是当其他函数组件调用了这个hook的时候会使hook的状态改变，需要使用闭包来就行缓存，还有一个问题是在使用多个hooks的时候需要使用链表来存储这些hooks，由于我自己还没有完全理解这些原理所以就不分享了。

## 如何在 Vue3 的函数组件中实现 React 式的函数组件 Hooks
不管我们写的是 SFC 组件还是 JSX 组件，最终会被编译成一个对象，这个对象上就有我们设置的 setup 方法，还有 render 方法，其中 SFC 组件的 render 是通过 template 模板编译出来的。然后再创建这个组件的虚拟DOM，再去渲染这个渲染 DOM，然后在渲染这个虚拟 DOM 的时候，如果是组件类型的虚拟 DOM 则需要创建组件实例，然后再执行组件实例上 render 方法获取组件的虚拟 DOM，然后再去渲染这个虚拟DOM，直到把所有的虚拟 DOM 渲染完毕，最后浏览器展示渲染的内容。  
 
React 是在每次执行函数组件方法之前，初始化 Hooks 的相关信息的，但我们是在 Vue3 框架之外去写的一个库，所以我们只能通过 Vue3 提供的 API 去实现我们的功能。首先我们可以通过 getCurrentInstance 这个 API 获取到当前函数组件的实例对象。我们是没办法在初始执行函数组件渲染之前对进行 Hooks 的初始化工作的，所以我们只能在获取 Hooks 的时候去进行 Hooks 的相关的初始化。

```
function updateWorkInProgressHook() {
  const instance = getCurrentInstance() as any;
  if (
    !currentlyRenderingFiber ||
    currentlyRenderingFiber.uid !== instance.uid
  ) {
    renderHooks(instance);
  }
    // ...
}
```

## 总结
我们想要在 Vue3 的函数组件上实现相同的功能，则是把 Hooks 的相关信息存储在对应的函数组件的实例对象上。另外在 Vue3 中如果需要将一个函数在组件渲染之后进行执行，则需要使用 watchEffect API，其中 options 的 flush 设置为 “post”，本质是渲染之后的 post 队列里面添加一个执行任务，从而达到跟 React 的 useEffect 的回调执行机制基本一致 。

实现的代码来自Cobyte
```
import { getCurrentInstance, watchEffect } from "vue";

// useLayoutEffect 的标记
const HookLayout = /*    */ 0b010;
// useEffect 的标记
const HookPassive = /*   */ 0b100;

// 当前的渲染的 Fiber 节点，对应 Vue 中则是当前渲染的组件函数的实例
let currentlyRenderingFiber: any = null;
// 当前正在工作的 Hook 节点
let workInProgressHook: any = null;
// 前一个 Hook
let currentHook: any = null;

// React 中启动一个 Fiber 协调的任务
function scheduleUpdateOnFiber(wip: any) {
  // 保存老 Fiber
  currentlyRenderingFiber.alternate = { ...currentlyRenderingFiber };
  renderHooks(wip);
  currentlyRenderingFiber.update();
}

// 初始化 Hooks 的相关设置
function renderHooks(wip: any) {
  currentlyRenderingFiber = wip;
  currentlyRenderingFiber.memorizedState = null;
  workInProgressHook = null;
}

// Hooks 设置
function updateWorkInProgressHook() {
  const instance = getCurrentInstance() as any;
  if (
    !currentlyRenderingFiber ||
    currentlyRenderingFiber.uid !== instance.uid
  ) {
    renderHooks(instance);
  }
  // alternate 是老 Fiber 的属性 
  const current = currentlyRenderingFiber.alternate;
  let hook;
  // 存在老的则是更新节点
  if (current) {
    currentlyRenderingFiber.memorizedState = current.memorizedState;
    if (workInProgressHook) {
      // 不是头节点
      hook = workInProgressHook = workInProgressHook.next;
      currentHook = currentHook.next;
    } else {
      // 头节点
      hook = workInProgressHook = current.memorizedState;
      currentHook = current.memorizedState;
    }
  } else {
    // 初始化
    currentHook = null;
    hook = {
      memorizedState: null,
      next: null,
    };

    if (workInProgressHook) {
      // 不是头节点
      workInProgressHook = workInProgressHook.next = hook;
    } else {
      // 头节点
      workInProgressHook = currentlyRenderingFiber.memorizedState = hook;
    }
  }

  return hook;
}

export function useState(initalState: any) {
  return useReducer(null, initalState);
}

export function useReducer(reducer: any, initalState: any) {
  // 获取 Hook
  const hook = updateWorkInProgressHook();

  if (!currentlyRenderingFiber.alternate) {
    hook.memorizedState = initalState;
  }
  // 通过 bind 方法进行缓存当前的组件函数的 Fiber 节点，Vue3 中则是函数组件的实例对象
  const dispatch = dispatchReducerAction.bind(
    null,
    currentlyRenderingFiber,
    hook,
    reducer
  );

  return [hook.memorizedState, dispatch];
}

function dispatchReducerAction(
  fiber: any,
  hook: any,
  reducer: any,
  action: any
) {
  hook.memorizedState = reducer ? reducer(hook.memorizedState) : action;
  // 调用 dispatch 的时候重新执行函数组件的渲染
  scheduleUpdateOnFiber(fiber);
}

function updateEffectImp(hookFlags: any, create: any, deps: any) {
  // 获取 Hook
  const hook = updateWorkInProgressHook();
  // 如果存在老 Hook 则进行对比
  if (currentHook) {
    const prevEffect = currentHook.memorizedState;
    if (deps) {
      const prevDeps = prevEffect.deps;
      if (areHookInputsEqual(deps, prevDeps)) {
        return;
      }
    }
  }
  const effect = { hookFlags, create, deps };
  hook.memorizedState = effect;

  invokeHooks(hookFlags, hook);
}

export function useEffect(create: any, deps: any) {
  return updateEffectImp(HookPassive, create, deps);
}

export function useLayoutEffect(create: any, deps: any) {
  return updateEffectImp(HookLayout, create, deps);
}

// 比较前后两个依赖是否发生变化
function areHookInputsEqual(nextDeps: any, prevDeps: any) {
  if (prevDeps === null) {
    return false;
  }

  for (let i = 0; i < prevDeps.length && i < nextDeps.length; i++) {
    if (Object.is(nextDeps[i], prevDeps[i])) {
      continue;
    }
    return false;
  }
  return true;
}

// 调用 Hooks
function invokeHooks(hookFlags: any, hook: any) {
  if (hookFlags & HookPassive) {
    postMessage(hook.memorizedState.create);
  } else if (hookFlags & HookLayout) {
    watchEffect(hook.memorizedState.create, { flush: "post" });
  }
}

// 通过 MessageChannel 创建一个宏任务
const postMessage = (create: any) => {
  const { port1, port2 } = new MessageChannel();
  port1.onmessage = () => {
    create();
  };
  port2.postMessage(null);
};
```



