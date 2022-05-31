---
title: react-hook-form为何如此受欢迎
date: 2022-05-31 10:28:20
categories: React  
tags:
- React
- 表单
---

## 背景
复杂表单在前端领域从来都是一个麻烦的问题，在前端三驾马车中，react由于天然缺少双向绑定的功能，所以在 react 中处理一些复杂的表单一直痛苦满满。  
有的朋友可能会问，antd用起来不香吗？我为什么还要费劲去熟悉另外一个库呢，而且这个库靠谱吗？毫无疑问antd是非常优秀的react组件库，它内置的Form组件在中后台项目中大规模使用。在 4.x 版本重构后，表单的写法也变得更加简洁。但如工程界一直讲的，在合适的场景使用合适的轮子，我们在下方具体阐述 react-hook-form 的优势以及在哪些场景，它会更香。

## 正文
一图胜千言，两个函数即可实现react表单的数据收集  
![这是图片](/img/531/demo.png "Magic Gardens")
### 拥抱 ref 与非受控组件
从这个图中，可以看到用极少量的代码就实现了对非受控表单组件的数据收集。无须在任何时刻都被迫使用受控组件，省去了react中颇为复杂的“双向绑定”。同时，它也支持 HTML 规范里的规则以及自定义校验。由于 react-hook-form 本身的体积非常小（压缩后大约只有 5KB），所以非常适合于轻量的、使用非受控组件的场景使用。
同时，react-hook-form 在受控组件中用起来也很方便：
```
import { useForm, Controller } from "react-hook-form";
import { Input } from "antd";

export default function Form() {
  const { control } = useForm({
    defaultValues: { id: "init" },
  });
  return (
    <div>
      <Controller name='id' as={Input} control={control} />
    </div>
  );
}
```
对比antdv3 的getFieldDecorator方案简洁了很多，与antdv4 对比，简洁度上也不分上下。而且由于 react-hook-form 是把组件的值保存在ref中的，因此会在组件内部变化时避免整个视图重绘，这样也会给大型表单项目带来可观的性能收益。在性能敏感的场景，一方面考虑依赖的体积，另一方面考虑交互的流畅，react-hook-form 不失为一个好方案。

### 动态增减表单项
在复杂表单开发中，常会遇到需要动态增减表单项的情况。比如，登记一组用户(users)，而每个 user 都有年龄(age)、名称(name)等属性，在用户填写表单的过程中，这个内容允许编辑、新增、删除、排序。react-hook-form 暴露了另外一个 custom-hooks——useFieldArray专门处理这个问题。
```
function Test() {
   const { control, register } = useForm();
   const { fields, append, prepend, remove, swap, move, insert } = useFieldArray({
      control,
      name: "test",
   });

   return (
      {fields.map((field, index) => (
      <input key={field.id} name={`test[${index}].value`} ref={register()} />
      ))}
   );
}
```
在 antd 3.x 版本中，需要额外维护一个保存组件 key 的数组，用 key 的增删代替组件的增删，十分繁琐。当然在 4.x 版本重写Form组件后，基本解决了这个问题，API 也与useFieldArray类似了。但是我个人在使用 antd 官方升级工具的过程中还是遇到了很多问题——从 3.x 升级到 4.x，大量的类型定义以及引用的路径都发生了变化。出于这点，基于typescript的 3.x 项目很难升级。所以在使用了 3.x 的老项目（支持 hooks）中，也可以考虑使用 react-hook-form 替换 antd 的Form。
 
### 强大的类型支持 
```
type Inputs = {
  example: string;
  exampleRequired: string;
};
function App() {
  const { register, handleSubmit, watch, error, getValues } = useForm<
    Inputs
  >();

  const onSubmit: SubmitHandler<Inputs> = (data) => {
    alert(JSON.stringify(data));
  };
  const example = watch("example"); // 类型推导为string;
  const values = getValues(); // 类型推导为Inputs
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <label>Example</label>
      <input name='example' defaultValue='test' ref={register} />
      <label>ExampleRequired</label>
      <input
        name='exampleRequired'
        ref={register({ required: true, maxLength: 10 })}
      />
      {errors.exampleRequired && <p>This field is required</p>}
      <input type='submit' />
    </form>
  );
}
```
这是非常令我心动的一点，表单对应的类型在整个传递过程中并没有丢失，在获取表单值的函数，如watch,getValues，setValue中还是保持了类型的正确定义。在typescript项目中，避免了更多 any 的出现，可以使你的项目尽可能保持严格的类型定义。这对于大型项目的可维护性来说很有帮助。

### 最快速的社区支持，没有之一
react-hook-form 的社区支持是我迄今为止见过最迅速的。开源项目经过一定版本的迭代达到稳定后，一般作者会更依赖于社区内部的回复，因为作者本人实在是分身乏术了，在 issue 列表里看到几个月甚至几年前未关闭的是家常便饭。所以当第一次看到 react-hook-form 的 issues 列表我都吃惊了，没有超过一周的未关闭的bug或是question。

## 结论
如果你正在寻找react的表单方案，react-hook-form 绝对值得一试。它的优势体现在这几个方面：
- 足够轻量，5kB的尺寸，以及使用ref保存值的特点，使它在性能敏感的场景有足够的吸引力。
- 对非受控组件的场景支持很好。
- 为动态变化的表单项提供了 custom-hooks，简洁的API帮助开发者应对复杂的场景。
- 强大的类型支持。
- 友善的社区和快速的支持。

同时，它的社区也依然在成长。如果希望能在开源项目中有所贡献/锻炼英语交流能力，它也不失为一个好地方。