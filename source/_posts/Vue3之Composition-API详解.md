---
title: Vue3之Composition API详解  
date: 2022-05-05 10:22:55  
cover: true  
categories: vue3  
tags:
  - vue3
  - Composition API
---

`Composition API`也叫组合式API，是Vue3.x的新特性。  

通俗的讲：  

没有`Composition API`之前vue相关业务的代码需要配置到option的特定的区域，中小型项目是没有问题的，但是在大型项目中会导致后期的维护性比较复杂，同时代码可复用性不高。Vue3.x中的composition-api就是为了解决这个问题而生的

compositon api提供了以下几个函数：
- setup
- ref
- reactive
- watcheffect
- watch
- computed
- toRefs
- 生命周期的hooks

## 一、setup组件选项
### 1. Props
- setup 函数中的第一个参数是 props。正如在一个标准组件中所期望的那样，setup 函数中的 props 是响应式的，当传入新的 prop 时，它将被更新
```
  export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```
但是，因为 props 是响应式的，你不能使用 ES6 解构，因为它会消除 prop 的响应性  
如果需要解构 prop，可以通过使用 setup 函数中的 toRefs 来安全地完成此操作。

### 2. 上下文
- 传递给 setup 函数的第二个参数是 context。context 是一个普通的 JavaScript 对象，它暴露三个组件的 property  
```
  export default {
  setup(props, context) {
    // Attribute (非响应式对象)
    console.log(context.attrs)

    // 插槽 (非响应式对象)
    console.log(context.slots)

    // 触发事件 (方法)
    console.log(context.emit)
  }
}
```
context 是一个普通的 JavaScript 对象，也就是说，它不是响应式的，这意味着你可以安全地对 context 使用 ES6 解构  

### 3. setup组件的 property
执行 setup 时，组件实例尚未被创建。因此，你只能访问以下 property：
- props
- attrs
- slots
- emit

换句话说，你将无法访问以下组件选项：
- data
- methods
- computed

### 4. ref reactive 以及setup结合模板使用
- ref用来定义响应式的 字符串、 数值、 数组、Bool类型
- reactive 用来定义响应式的对象

### 5. 使用 this
- 在 setup() 内部，this 不会是该活跃实例的引用，因为 setup() 是在解析其它组件选项之前被调用的，所以 setup() 内部的 this 的行为与其它选项中的 this 完全不同。这在和其它选项式 API 一起使用 setup() 时可能会导致混淆

## 二、toRefs - 解构响应式对象数据
- 把一个响应式对象转换成普通对象，该普通对象的每个 property 都是一个 ref ，和响应式对象 property 一一对应
```
  <template>
<div>
    <h1>解构响应式对象数据</h1>
    <p>Username: {{username}}</p>
    <p>Age: {{age}}</p>
</div>
</template>

<script>
import {
    reactive,
    toRefs
} from "vue";

export default {
    name: "解构响应式对象数据",
    setup() {
        const user = reactive({
            username: "张三",
            age: 10000,
        });

        return {
            ...toRefs(user)
        };
    },
};
</script>
```

## 三、computed - 计算属性
```
  <template>
<div>
    <h1>解构响应式对象数据+computed</h1>

    <input type="text" v-model="firstName" placeholder="firstName" />
    <br>
    <br>
    <input type="text" v-model="lastName" placeholder="lastName" />

    <br>
    {{fullName}}
</div>
</template>

<script>
import {
    reactive,
    toRefs,
    computed
} from "vue";

export default {
    name: "解构响应式对象数据",
    setup() {
        const user = reactive({
            firstName: "",
            lastName: "",
        });

        const fullName = computed(() => {
            return user.firstName + " " + user.lastName
        })

        return {
            ...toRefs(user),
            fullName
        };
    },
};
</script>
```

## 四、readonly “深层”的只读代理
- 传入一个对象（响应式或普通）或 ref，返回一个原始对象的只读代理。一个只读的代理是“深层的”，对象内部任何嵌套的属性也都是只读的
```
  <template>
  <div>
    <h1>readonly - “深层”的只读代理</h1>
    <p>original.count: {{original.count}}</p>
    <p>copy.count: {{copy.count}}</p>
  </div>
</template>

<script>
import { reactive, readonly } from "vue";

export default {
  name: "Readonly",
  setup() {
    const original = reactive({ count: 0 });
    const copy = readonly(original);

    setInterval(() => {
      original.count++;
      copy.count++; // 报警告，Set operation on key "count" failed: target is readonly. Proxy {count: 1}
    }, 1000);


    return { original, copy };
  },
};
</script>
```

## 五、watchEffect
- 在响应式地跟踪其依赖项时立即运行一个函数，并在更改依赖项时重新运行它。
```
  <template>
<div>
    <h1>watchEffect - 侦听器</h1>
    <p>{{data.count}}</p>
    <button @click="stop">手动关闭侦听器</button>
</div>
</template>

<script>
import {
    reactive,
    watchEffect
} from "vue";
export default {
    name: "WatchEffect",
    setup() {
        const data = reactive({
            count: 1,
            num: 1
        });
        const stop = watchEffect(() => console.log(`侦听器：${data.count}`));
        setInterval(() => {
            data.count++;
        }, 1000);
        return {
            data,
            stop
        };
    },
};
</script>
```

## 六、watch 、watch 与watchEffect区别
对比watchEffect，watch允许我们:
- 懒执行，也就是说仅在侦听的源变更时才执行回调；
- 更明确哪些状态的改变会触发侦听器重新运行；
- 访问侦听状态变化前后的值

## 七、组合式api生命周期钩子
![这是图片](/img/505/hook.png "Magic Gardens")  
- 因为 setup 是围绕 beforeCreate 和 created 生命周期钩子运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 setup 函数中编写

## 八、Provider Inject
没用过......



