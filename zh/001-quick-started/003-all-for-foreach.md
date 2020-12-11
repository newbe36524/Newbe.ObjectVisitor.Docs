---
title: 0x03-ForEach 全面观
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

前面，我们已经了解组成一个 object visitor 最基本的部件以及最佳的性能做法。本篇我们来介绍一下更多关于 `ForEach` 方法的奇怪操作。

## ForEach 的重载

ForEach 以下主要的重载形式：

```cs
ForEach(Expression<Action<IObjectVisitorContext<T, object>>> foreachAction)
ForEach(Expression<Action<string, object>> foreachAction)

ForEach<TValue>(Expression<Action<IObjectVisitorContext<T, TValue>>> foreachAction)
ForEach<TValue>(Expression<Action<string, TValue>> foreachAction)
```

这四个重在分别按照“泛型”和“使用 Context”两两交叉得到。

### 使用 Context ForEach 重载

首先说明“使用 Context”重载所带来的区别。

在下面这个普通重载中，你只能获取 name 和 value 两个属性。

```cs
ForEach(Expression<Action<string, object>> foreachAction)
```

这也是最基本的重载形式，可以让你读取所有属性的名称和值。

但是你可能还需要更多的上下文信息，因此就引入了一个可以使用 Context 进行访问的重载形式：

```cs
ForEach(Expression<Action<IObjectVisitorContext<T, object>>> foreachAction)
```

这就带来了一些优势：

- 你可以访问除了 Name 和 Value 之外更多的信息了，目前至少包括：表示原始访问对象的 SourceObject 和表示属性信息的 PropertyInfo
- Context 不是完全只读的。其中的 Value 属性允许赋值。因此，你可以通过赋值来修改原始对象中的属性值。这是通过参数传递无法做到的(未使用 ref)。
- 方法签名不会发生变化，因为只有一个参数。不会因为增加属性而需要修改方法签名。

不过这也会带来一些损失：

- 方法签名更长
- 运行时会创建 Context 对象，这将带来十分轻微的性能开销

因此，如果在你的场景中以上优势大于其损失，不妨也考虑使用 Context 。

### 使用泛型 ForEach 重载

使用 ForEach 的泛型重载可以使得得到的 Value 是一个明确类型的属性值，而不会是一个个 object 类型。

这样做，你可以得到以下这些好处：

- 对于值类型你可以避免装箱和拆箱消耗
- 你可以明确使用预先知道的类型

当然，你也就失去了一些：

- 你无法通用的处理所有属性，因为特别是一个类型中属性类型各不相同的时候

在使用泛型重载时，还有一点非常重要的内容，就是对需要访问的类型进行过滤。毕竟一个对象中并不是所有的类型都可以装换为同一个泛型类型。我们将会在下一小节进行说明。

Context 也有泛型版本的重载。其作用也就是结合了 Context 和泛型。

## 添加扩展数据

和常规的重载类似，如果一个 object visitor 被指定了需要传递扩展数据。那么其重载也将发生变化，可以对传入的扩展数据进行处理：

```cs
ForEach(Expression<Action<IObjectVisitorContext<T, TExtend, object>>> foreachAction)
ForEach(Expression<Action<string, object, TExtend>> foreachAction)

ForEach<TValue>(Expression<Action<IObjectVisitorContext<T, TExtend, TValue>>> foreachAction)
ForEach<TValue>(Expression<Action<string, TValue, TExtend>> foreachAction)
```

这点开发者们应该非常熟悉，因为我们前篇中使用的“格式化对象”示例中就使用的是带有扩展数据的 object visitor。

和常规重载类似，以上重载也有 context 形式和泛型形式，功能也是类似的。这里也就不多说明。

## 多次调用 ForEach

在一个 object visitor 中允许多次调用 ForEach 进行注册。

值得注意的是，这些注册相互之间是共享同一原始对象的，因此，前面一个 ForEach 的修改是会对后一个产生影响的。

这里有一个例子，我们将 order 中的字符串加入到 list1 当中。但是在第一次添加之后会将其属性值修改为了空。因此 list2 就无法得到数值了。

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Newbe.ObjectVisitor;

namespace yueluo_dalao_yes
{
    class Program
    {
        static void Main(string[] args)
        {
            var order = new OrderInfo
            {
                Buyer = "yueluo",
            };
            var list1 = new List<object>();
            var list2 = new List<object>();
            order.V()
                .ForEach<string>(c => PutToList(c, list1))
                .ForEach<string>(c => PutToList(c, list2))
                .Run();
            Console.WriteLine(list1.Count);
            Console.WriteLine(list2.Count);
        }

        static void PutToList(IObjectVisitorContext<OrderInfo, string> c, List<object> list)
        {
            if (!string.IsNullOrEmpty(c.Value))
            {
                list.Add(c.Value);
                c.Value = "";
            }
        }
    }

    public class OrderInfo
    {
        public int OrderId { get; set; }
        public string Buyer { get; set; }
        public decimal TotalPrice { get; set; }
    }
}
```

这段代码将会输出以下结果：

```cs
1
0
```
