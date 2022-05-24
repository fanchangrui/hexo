---
title: react-antd table展开行实现以及自定义和踩坑
date: 2022-05-24 13:36:57  
cover: true  
categories: React
tags:
- React
- antd
---

## 前言
最近在做一个权限管理的后台项目，经常涉及到树形数据展示，就用到了antd，table表格树形数据组件。记录一下使用过程的坑。  
## 环境及配置
- antd -3.23.6(大版本为antd 3)
- antd 3和antd 4版本对于表格来说变动很小只是把展开配置放到了一个expandable的对象里，所以只需要稍微变动就可以运用到antd v4版本  

## antd Table树形数据展示及使用
使用很简单只需要数据是树形结构的就可以（一般都是children为标志，如果不是可以使用childrenColumnName树形指定key名，api文档里写的很清楚）。  
- 1.自定义展开收起图标：
  业务要求需要自定义展开收起（默认的不太好看）  
  实现思路：
  利用 expandIcon，判断是否可以展开来实现  
```
import React, { Component } from 'react';
import { Table, Divider, Tag,Icon } from 'antd';
const data=[
    {
        name:"破阵",
        age:"32",
        tel:"5555",
        leve:1,
        children:[
            {
                name:"山鬼谣",
                age:"18",
                tel:"55554",
                leve:2,
                children:[
                    {
                        name:"弋痕夕",
                        age:"34",
                        tel:"55554",
                        leve:3,
                    }
                ]
            },
            {
                name:"天净沙",
                age:"45",
                tel:"555549",
                leve:2,
            }
        ]
    },
    {
        name:"辗迟",
        age:"18",
        tel:"5555",
        leve:1,
        children:[]
    }
];
const coumns=[
    {
        title: 'name',
        dataIndex: 'name',
        key: 'name',
        render: text => <a>{text}</a>,
    },
    {
        title: 'age',
        dataIndex: 'age',
        key: 'age',
        render: text => <a>{text}</a>,
    },
    {
        title: 'tel',
        dataIndex: 'tel',
        key: 'tel',
        render: text => <a>{text}</a>,
    },
];
class index extends Component {
    render() {
        return (
            <div>
                <Table dataSource={data} columns={coumns} 
                    expandIcon={({expanded, onExpand, record})=>{ //expanded-是否可展开, onExpand-展开事件默认, record-每一项的值 设置自定义图标
                        if(expanded){//根据是否可以展开判断
                            return  <Icon type="minus-circle" onClick={e => onExpand(record, e)}/>
                        }else{
                            return  <Icon type="plus-circle" onClick={e => onExpand(record, e)}/>
                        }
                    }}
                    />
            </div>
        );
    }
}
export default index;
```
- 2.业务要求默认展开
  要知道antd里只要和default沾边的都只是初次渲染管用（比如给input设置默认值等等），所以官方给的是expandedRowKeys和onExpand结合实现。  
  实现思路：  
  expandedRowKeys和onExpand结合实现。判断是否展开，动态改变expandedRowKeys。  
  注意数据里的key必须是唯一的，否则会出现展开一个另一个也跟着展开（当然如果需求就是联动可以相同）。  
  先加上expandedRowKeys默认是展开了，但是会出现点击展开和折叠就不生效了，所以要结合onExpand实现（使用这个默认的就不生效了，默认没有子项是不展示图标的，所以还要加一个判断）  
```
class index extends Component {
    state = {
        expandedRowKeys: []
    }
    componentDidMount() {
        let keyArr = [];
        data.map((item, index) => { //这里就可以把要展开的key加进来记住必须是唯一的
            keyArr.push(item.key)
        })
        this.setState({
            expandedRowKeys: keyArr
        })
    }
    onExpand = (expanded, record) => { //expanded是否展开  record每一项的值
        let { expandedRowKeys } = this.state;
        if (expanded) {
            let arr = expandedRowKeys;
            arr.push(record.key);
            this.setState({
                expandedRowKeys: arr
            })
        } else {
            let arr2 = [];
            if (expandedRowKeys.length > 0 && record.key) {
                arr2 = expandedRowKeys.filter((key) => {
                    return key !== record.key;
                })
            }
            this.setState({
                expandedRowKeys: arr2
            })
        }
    }
    render() {
        return (
            <div>
                <Table dataSource={data} columns={coumns}
                    expandIcon={({ expanded, onExpand, record }) => { //expanded-是否可展开, onExpand-展开事件默认, record-每一项的值 设置自定义图标
                        if (record.children&&record.children.length != 0) {//有子项加才有图标（根据自己的额业务需求变动）
                            if (expanded) {//根据是否可以展开判断  
                                return <Icon type="minus-circle" onClick={e => onExpand(record, e)} />
                            } else {
                                return <Icon type="plus-circle" onClick={e => onExpand(record, e)} />
                            }
                        } else {
                            return ''
                        }
 
                    }}
                    expandedRowKeys={this.state.expandedRowKeys}
                    onExpand={this.onExpand}
                />
            </div>
        );
    }
}
```
- 3.给不同层级加颜色（类似于隔行变色）
  实现思路：根据level来判断层级利用rowClassName来动态该改变类名  
```
class index extends Component {
    state = {
        expandedRowKeys: []
    }
    componentDidMount() {
        let keyArr = [];
        data.map((item, index) => { //这里就可以把要展开的key加进来记住必须是唯一的
            keyArr.push(item.key)
        })
        this.setState({
            expandedRowKeys: keyArr
        })
    }
    onExpand = (expanded, record) => { //expanded是否展开  record每一项的值
        let { expandedRowKeys } = this.state;
        if (expanded) {
            let arr = expandedRowKeys;
            arr.push(record.key);
            this.setState({
                expandedRowKeys: arr
            })
        } else {
            let arr2 = [];
            if (expandedRowKeys.length > 0 && record.key) {
                arr2 = expandedRowKeys.filter((key) => {
                    return key !== record.key;
                })
            }
            this.setState({
                expandedRowKeys: arr2
            })
        }
    }
    classNameFn=(record, index)=>{
      let className="";
      if(record){
          if(record.leve==1){
            className="leve1"
          }else if(record.leve==2){
            className="leve2"
          }else if(record.leve==3){
            className="leve3"
          }else{
            className=""
          }
      }
      console.log(className)
      return className;
    }
    render() {
        return (
            <div>
                <Table dataSource={data} columns={coumns}
                    expandIcon={({ expanded, onExpand, record }) => { //expanded-是否可展开, onExpand-展开事件默认, record-每一项的值 设置自定义图标
                        if (record.children&&record.children.length != 0) {
                            if (expanded) {//根据是否可以展开判断  
                                return <Icon type="minus-circle" onClick={e => onExpand(record, e)} />
                            } else {
                                return <Icon type="plus-circle" onClick={e => onExpand(record, e)} />
                            }
                        } else {
                            return ''
                        }
 
                    }}
                    expandedRowKeys={this.state.expandedRowKeys} //展开行
                    onExpand={this.onExpand} //展开事件
                    rowClassName={this.classNameFn} //表格行类名
                />
            </div>
        );
    }
}
```
```
@import '~antd/es/style/themes/default.less';
.leve1{
  background: burlywood;
}
.leve2{
    background: blue;
}
.leve3{
    background: gray;
```
- 4.给表格设置样式
  给表格设置样式需要元素审查一级一级的找  
  利用：'：global'来覆盖就行  
## 总结
antd是很好的开源UI库，但也存在很多问题。需要根据自己的业务来，选择适合的组件。另外antd-pro是一个不错的框架（dva+umi），可以快速开发应用。

