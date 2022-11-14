---
title: Quark Design和Web Components
date: 2022-11-14 13:36:31
categories:  Web Components
tags:
- Web Components
- Quark Desigen
---

## 前言
前几天哈罗单车前端团队开源了Quark Design组件库，号称是下一代的前端组件库以至于可以在任何框架或原生js中使用。  
看到这的时候我以为是类似于Ant Desigen of Vue 和Ant Design合起来然后按照项目框架分别引用😂，然后仔细一看好像是真的支持任何框架依靠Web Components来实现，关于Web Components我之前只是听说过而一直没怎么去了解，趁这个框架我去学习了Web Components。

## Web Components是什么
Web Components与React、Vue等框架中的组件类似，其实在Vue中也采用了很多基于Web Components的设计。这是一个可复用的UI构建模块，封装了所有渲染所需要的HTML、CSS以及基于Javascript的逻辑。最大的区别是，它不依赖于特定的JavaScript框架，而是利用浏览器原生提供的技术，这样你的Web组件就与框架无关了。

## Web Components的主要内容
Web Components的主要内容如下：  
- Custom Elements：原生提供的API，用于可自定义Custom Elements和行为。
- Shadow DOM：原生提供的API，用于封装与主文档DOM隔离的私有DOM，不受外部DOM的样式和行为的影响。
- Templates
- Attributes
- Slots
- Life cycle methods

## Custom Elements
Custom Elements是Web Components中的一个重要特性，可以提供给开发者将html的功能封装为自定义标签，方便进行复用。也就是说，Custom Elements本质上是用户用于实现在html中有效使用的，类似div、button等自定义标签。最大区别在于，自定义标签有着自己的模板、行为和标签名称，通过Javascript的API实现的。  
Custom Elements总是使用连字符-进行自定义标签的标识，即one-button，各大浏览器公司达成公司不在创建任何新的标签元素，来防止元素冲突。熟悉vue框架的开发者应该知道，在vue中的自定义组件命令有OneButton或one-button这种结构，官方推荐的是one-button这种使用方式。  
我们来简单实现一下一个自定义标签：
```
// one-button.js
// 1、创建自定义组件的类
class OneButton extends HTMLElement{
    constructor(){
        super()
        // 2、创建组件内容
        this.innerHTML = `<button>hello onechuan</button>`
    }
}

// 3、进行自定义组件的注册
customElements.define("one-button", OneButton)

// index.html
// 4、html中进行使用
<one-button></one-button>
<script src="./one-button.js"></script>
```

## Shadow DOM
Shadow DOM与原生DOM的区别就是用户自定义封装的DOM，本质上还是DOM元素。不同的是，Shadow DOM可以将自定义的DOM片段与其他DOM进行隔离，可以实现样式不受外部DOM的影响，类似与使用iframe内嵌了一个html。也正是Shadow DOM的存在，能够进行DOM隔离、实现独立的组件王国，使得自定义组件跨域框架的约束，不再耦合，确保在任何无框架和任何框架中正常使用。
![这是图片](/img/1114/1.png "Magic Gardens")  
Shadow DOM 并不是新生事物，在原生video标签中其实就有内置的Shadow DOM ，包含一系列的按钮和其他控制器等。而Web Components只不过是允许用户根据需要使用Shadow DOM来实现自定义标签。
```
// one-button.js
// 1、创建自定义组件的类
class OneButton extends HTMLElement{
    constructor(){
        super()
        // 2、创建shadow dom
        this.attachShadow({mode: 'open'})
        this.shadowRoot.innerHTML = `<button>hello onechuan</button>`
    }
}

// 3、进行自定义组件的注册
customElements.define("one-button", OneButton)
```
在上面运行结果中，实现的展示效果是一样，但是渲染的dom节点却有所不同，在使用shadow dom实现的标签中会有#shadow-root字样标识。


## Template
见到template模板，使用过vue框架的开发者就再熟悉不过了，template标签本身不会在html中进行立即渲染，而是用于对待使用的代码进行标记，在需要被使用的时候才会进行渲染。也正是因为template的特性，使得我们可以将一些可重复使用的代码用template进行组织，通过javascript获取它的引用，再添加到DOM中。
```
<one-button></one-button>
<template>
    <button>hello yichuan</button>
</template>
<script>
    const template = document.createElement("template")
    template.innerHTML = `<button>hello onechuan</button>`
    // 1、实现自定义组件的类
    class OneButton extends HTMLElement{
        // 2、定义自定义组件的内容
        constructor(){
            super()
            // 2-1、创建影子DOM
            this.attachShadow({mode: "open"})
            // 2-2、设置影子DOM的内容
            
            this.shadowRoot.appendChild(template.content.cloneNode(true))
        }
    }
    // 3、注册自定义组件
    customElements.define("one-button", OneButton)
</script>
```
在上面代码片段中，可以看到在需要使用的时候将模板内容追加到shadow dom中，即模板内容不会立即被渲染，而是在被调用后才会在页面进行渲染。

## Slots
插槽由其name属性标识，并且允许您在模板中定义占位符，当在标记中使用该元素时，该占位符可以填充所需的任何 HTML 标记片段，这个和Vue的slot差不多功能和实现。
```
<one-button>
    <button>hello yichuan</button>
</one-button>
<script>
    const template = document.createElement("template")
    template.innerHTML = `
        <div>
            <slot><button>hello onechuan</button></slot>
        </div>
    `
    class OneButton extends HTMLElement{
        constructor(){
            super()
            this.attachShadow({mode:"open"})
            this.shadowRoot.appendChild(template.content.cloneNode(true))
        }
    }
    customElements.define("one-button",OneButton)
</script>
```

## Life Cycle Methods
这里有三个方法分别是：
- connectedCallback：会在自定义组件插入DOM的首次渲染时调用一次。此时 shadow dom已经挂载到DOM树上，对此可以通过connectedCallback方法来访问shadow dom上的属性、子元素以及监听事件。倘若组件被移除或被移动，将会再次被调用。
- attributeChangedCallback：监听custom element自身属性的增删改，当发生状态改变时调用此方法，在使用前必须在一个observedAttributes() 的静态方法中定义要观察的属性。该方法返回一个属性名称的数组。一旦observedAttributes返回了属性值数组，则attributeChangedCallback方法会在每次该属性变化时调用。
- disconnectedCallback：当 custom element 从文档 DOM 中删除时，disconnectedCallback被调用。

## 总结
Quark Design就是类似使用上述官方api实现的前端组件库，基本可以在任何项目中重复使用。我们在在构建Web Components时，步骤如下：
- 首先，必须使用customElements.define()注册自定义元素
- 构造函数会调用，将节点添加到shadow dom中
- 然后，进行一次初始化，执行connectedCallback方法
- 当元素的属性被更新时，它的attributeChangedCallback()被触发
- 当一个事件从元素中被移除时，disconnectedCallback()方法被调用以清理事件监听器
- shadow dom - 是DOM的一个封装版本，用于防止CSS泄露
- Slots - 用于添加和访问自定义元素组件的子元素，也有一种特殊的方式来访问使用命名为Slot的子元素

因为第一次了解Web Components这个内容，所以上述介绍很浅显也借鉴了许多文档资料，如有错误请指正。
