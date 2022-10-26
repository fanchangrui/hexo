---
title: Ant Desigin的tree控件使用过程中的一些踩坑及解决方法
date: 2022-10-26 13:34:15
categories:  React
tags:
- Antd
- React
- tree
---

## 前言
最近一段时间因为工作太忙的原因导致一直没更新，今天空下来准备给大家分享一下这段时间内在用antd的tree组件在权限控制中的一些坑以及是如何解决的。

## 权限设计思路 
一般的权限分配其实实现起来很简单，无非是利用tree组件接收数据和修改节点id，我做的这个权限接口返回了两个数据：一、有权限的id数组，二：权限的树形结构数据。  
那到树形结构的数据后利用treeNode渲染树，具体实现如下  
```
    renderTreeNodes = data =>
        data.map(item => {

            if (item.children) {
            return (
                <TreeNode title={item.title} key={item.id}>
                {this.renderTreeNodes(item.children)}
                </TreeNode>
            );
            }
            return <TreeNode title={item.title} key={item.id}></TreeNode>;
    })
```
然后把需要勾选的id数组绑定checkKeys，最后设置勾选事件onCheck改变需要传给后端的数组id，树组件默认开启受控。  
没错理论上权限功能已经全部实现，就是这么简单，但这需要后端去判断后续的所有逻辑，果然bug来了。

## 勾选了子节点然后父节点没权限？
一段时间后测试提出了bug：勾选了子菜单的权限但是没有选择全部子菜单后，父菜单的入口也没了，然后我查看了这个步骤后到底给后端传了什么数据，我发现如果只勾选部分子节点的话，父节点处于半勾选状态，这样子不会传入父节点的id，于是乎我和后端开始互相甩锅，我说你为啥不能判断他是否有父节点有的话给父节点加上权限，后端说你为什么不能把父节点传过来，最后我斗争失败决定前端来改。  
解决方法就在onCheck事件中重新设置一个变量来push父节点的id：  
```
    onCheck = (checkedKeys,info)=>{
        const curKeys=this.state.keys;
        this.state.keys=curKeys.concat(info.halfCheckedKeys);
        let checkedKeysLast =checkedKeys.concat(info.halfCheckedKeys)
        this.setState({checkedKeysLast})
        this.state.checkedKeysLast =checkedKeysLast
        this.setState({ checkedKeys });
        this.state.checkedKeys=checkedKeys;

    }
```
当选择子节点的时候把他的所有父节点也传入checkedKeysLast，但是注意不能改变渲染的原id数组，因为这样会全选，最后把新设置的数组传给接口。以为这样子就好了吗？马上发现了一个新bug。  

## 所有权限都变成全选了！
然后我勾选了一个字节点后，发现的确父菜单可以访问了，但当我测试第二次的时候发现所有节点都被勾选了，但是实际上那些节点的功能都没有，我检查了一下发现是因为接口返回的数组都包含父节点，然后受控组件父节点选择的话会自动全部勾选子节点导致渲染问题。  
我思考了下这个问题有两个解决思路，一个是改为非受控组件自己写check的实现方法，一个是去除原数组里的所有父节点。因为第一种方法改动太大而且可能会导致其他的bug所以实现第二种思路。

## 最后解决方法
我一开始准备循环checkedKeys的每一个id到treeData中判断是否是最底层的子节点然后进行删除，然后我发现这个方法太麻烦了，又想到了直接筛选treeData所有子节点然后和checkedKeys对比合并一个两个数组都有相同id的新数组。
```
      const checkKeys = this.state.checkedKeys.concat(this.props.checkedKeys);
        const check = (data, list) => {
            data.forEach((item) => {
                if (item.children && item.children.length > 0) {
                    check(item.children, list);
                } else {
                    list.push(item.id);
                }
            });
            return list;
        }
        const checkData = (data) => {
            const list = [];
            return check(data, list);
        };
        let sonCheckKeys =checkData(this.state.treeData)
        const resultValue =(arr1,arr2) =>{
            console.log(arr1,arr2)
            let newArr = arr1.filter(t => arr2.includes(t));
            return newArr;
        }
        const checkKeysFin =resultValue(checkKeys,sonCheckKeys)
        this.state.checkedKeys =checkKeysFin
        this.setState({checkKeys:checkKeysFin})
```

## 写在最后
其实这种bug前后端都可以改，就看你能不能让对方退步了。