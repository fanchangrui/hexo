---
title: element ui日期选择器获取初始值后无法修改
date: 2023-03-24 15:02:26
categories:  Vue
tags:
- Vue
- element
---


最近在element-ui的日期选择器中，获取初始日期的开始时间和结束时间后，无法修改时间、删除等操作，代码如下：
```
    <el-form-item
                label="活动时间："
                prop="dateRange"
                >
                <el-date-picker
                    :disabled="isDetail"
                    v-model="form.dateRange"
                    size="small"
                    format="yyyy-MM-dd HH:mm:ss"
                    value-format="yyyy-MM-dd HH:mm:ss"
                    type="datetimerange"
                    start-placeholder="创建开始日期"
                    end-placeholder="创建结束日期"
                ></el-date-picker>
            </el-form-item>   

              if (this.$route.query.id ||  this.$route.query.detailId) {
                getDetail({ id: this.$route.query.id || this.$route.query.detailId })
                .then(res => {
                    this.form = res.data;
                    this.form.dateRange[0] = res.data.startTime
                    this.form.dateRange[1] = res.data.endTime


                })
                .catch(() => {});
            }
```
然后我尝试change事件打印了dateRange的值，已经显示修改了，但是视图却一直不更新，我怀疑是这个对象失去了响应式，我想到是否可以手动给这个变量加响应式，代码如下：  
```
 if (this.$route.query.id ||  this.$route.query.detailId) {
                getDetail({ id: this.$route.query.id || this.$route.query.detailId })
                .then(res => {
                    this.form = res.data;
                    this.$set(this.form, 'dateRange', [res.data.startTime, res.data.endTime])


                })
                .catch(() => {});
            }

```
然后这个bug就解决了，为什么会出现这个bug呢，我怀疑是vue的data数据里如果是一个二级对象是Object或者Array的话，他下一级的数据会失去响应式，需要用$set手动给这个变量加上响应式，或者把这个变量定义在data的第一层。  
实际上这些bug无法解决很多都是对于Vue原理不够了解导致的，我们可以平时多看看源码了解下Vue响应式原理就可以在出现bug的时候迅速定位了。