---
title: 0x02-创建并缓存 Object Visitor
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## 一切都是为了更加高效

前篇，我们通过一个简单的实例来介绍了如何使用 Object Visitor 来将 `OrderInfo` 的所有属性连接并输出。

虽然效果已经实现了，但是为了简化篇幅还是存在一些性能不佳的实践，本节我们就来介绍使用 Newbe.ObjectVisitor 真正完整的姿势。

## 缓存 Object Visitor

上节的核心代码如下：

```cs
var sb2 = new StringBuilder();
var visitor = order.V()
    .ForEach((name, value) => sb2.AppendFormat("{0}: {1}{2}", name, value, Environment.NewLine));
visitor.Run();
Console.WriteLine(sb2.ToString());
```

这段代码中，我们直接创建了一个 object visitor ，然后在后续直接使用它，接着方法就直接结束了。

对于一次性使用的 object visitor 来说，这没有问题。但是如果这段代码位于常用的热点路径上，这样重复创建 object visitor 的方式无疑是一种浪费。因此，我们可以考虑缓存 object visitor 在静态字段中。

于是，我们就得到了下面这段代码：

```cs
using System;
using System.Text;
using Newbe.ObjectVisitor;

namespace yueluo_dalao_yes
{
    class Program
    {
        private static readonly StringBuilder _sb = new StringBuilder();
        private static readonly ICachedObjectVisitor<OrderInfo> _visitor = default(OrderInfo).V()
                .ForEach((name, value) => _sb.AppendFormat("{0}: {1}{2}", name, value, Environment.NewLine))
                .Cache();

        static void Main(string[] args)
        {
            var order = new OrderInfo
            {
                OrderId = 1,
                Buyer = "yueluo",
                TotalPrice = 62359.1478M
            };

            _visitor.Run(order);
            Console.WriteLine(_sb.ToString());
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

这里，我们做了这些改动：

1. 将 object visitor 修改了为一个静态字段，并且使用字段初始化进行赋值。
2. 初始化 object visitor 时，由于此时 `OrderInfo` 对象还没有确定，因此使用 `default(OrderInfo)` 来表示 object visitor 需要访问的目标类型。
3. 我们在 `ForEach` 的后面，调用了 `Cache` 方法来创建一个缓存好的 object visitor。这里结合实现的细节这里可以做出解释： `Cache` 实际上就是将表达式编译为委托，这样后续就可以反复使用这个委托，达到最好的性能。
4. 由于是在静态字段中初始化，所以 `ForEach` 中的 `StringBuilder` 对象也必须是一个静态字段，因此我们也将其定义在 `_sb` 字段中。
5. 通过上面的改造，我们就可以直接使用被缓存好的 `_visitor` 来创建格式化字符串了。

## 你还需要 Extend Object

上节，我们通过简单的改造，将 object visitor 缓存了起来，避免了多次创建 object visitor 的开销。

但是，遗留了一个非常严重的问题：`ForEach` 当中直接使用了静态字段 `_sb` 进行字符串拼接，这会导致，多次调用 `_visitor.Run(order)` 的话会不断的在原始基础上拼接。这显然很不合理。

为了应对在 object visitor 运行过程中可能除了对被访问对象（`order`）进行处理之外，还需要一个额外的对象（`StringBuilder`），我们增加了一个称之为 Extend Object 的特性。

接下来，我们来改造一下上节的样例：

```cs
using System;
using System.Text;
using Newbe.ObjectVisitor;

namespace yueluo_dalao_yes
{
    class Program
    {
        private static readonly ICachedObjectVisitor<OrderInfo, StringBuilder> _visitor = default(OrderInfo).V()
                .WithExtendObject<OrderInfo, StringBuilder>()
                .ForEach((name, value, sb) => sb.AppendFormat("{0}: {1}{2}", name, value, Environment.NewLine))
                .Cache();

        static void Main(string[] args)
        {
            var order = new OrderInfo
            {
                OrderId = 1,
                Buyer = "yueluo",
                TotalPrice = 62359.1478M
            };

            var sb1 = new StringBuilder();
            _visitor.Run(order, sb1);
            Console.WriteLine(sb1.ToString());

            var sb2 = new StringBuilder();
            _visitor.Run(order, sb2);
            Console.WriteLine(sb2.ToString());
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

这里，我们做了这些改动：

1. 通过 `WithExtendObject` 扩展方法，指定了我们的 object visitor 在运行的时候需要传入一个 `StringBuilder` 类型作为 Extend Object。
2. 修改了 `ForEach` 来接收外部传入的 `StringBuilder`，并且调用其拼接方法。
3. 修改了 `_visitor.Run` 的调用方法。因为，此时不仅仅需要传递 `order`，还需要传递一个 `StringBuilder` 作为 Extend Object。
4. 运行这个样例，你就会发现输出了两遍一样的信息，这正是我们想要的。

通过 Extend Object，我们就可以在 `ForEach` 中接受一些额外的参数以便实现一些逻辑。

## 我们还可以再优化一下

我们可以使用泛型静态类来缓存我们的 object visitor，这样我们就可以实现所有类型的属性拼接功能：

```cs
using System;
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
                OrderId = 1,
                Buyer = "yueluo",
                TotalPrice = 62359.1478M
            };

            Console.WriteLine(order.FormatToString());
        }
    }

    public static class FormatToStringExtensions
    {
        public static string FormatToString<T>(this T obj)
        {
            var sb = new StringBuilder();
            FormatStringVisitor<T>.Instance.Run(obj, sb);
            var re = sb.ToString();
            return re;
        }

        private static class FormatStringVisitor<T>
        {
            internal static readonly ICachedObjectVisitor<T, StringBuilder> Instance = CreateFormatToStringVisitor();

            private static ICachedObjectVisitor<T, StringBuilder> CreateFormatToStringVisitor()
            {
                var re = default(T)!
                    .V()
                    .WithExtendObject<T, StringBuilder>()
                    .ForEach((name, value, s) => s.AppendFormat("{0}:{1}{2}", name, value,
                        Environment.NewLine))
                    .Cache();
                return re;
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

这里，我们做了这些改动：

1. 我们增加了一个 `FormatToStringExtensions` 静态类来添加一个扩展方法 `FormatToString`，这样所有的类型对象就都具备了 `FormatToString` 这个方法。
2. 增加了一个 `FormatStringVisitor<T>` 泛型静态类，用它来创建并保存一个 object visitor 实例到 `Instance` 属性，这样每个类型 T 就有了各自独立的 object visitor。
3. 在需要格式化的地方调用 `FormatToString` 方法，便可以完成先前需要实现的相同逻辑。

这就是最终的一个完全体方案。实际上，`FormatToString` 方法目前已经在 Newbe.ObjectVisitor 包中内置存在了，其实现方法就如上文所述：

[点击此处可以查看 FormatToStringExtensions 的源码](https://github.com/newbe36524/Newbe.ObjectVisitor/blob/main/src/Newbe.ObjectVisitor/Newbe.ObjectVisitor/Util/FormatToStringExtensions.cs)

## 本篇小结

通过本篇，我们通过一系列优化得到了使用 object visitor 的最佳范式。

通过最佳范式已经能够在大多数的场景中得以适用。

当然，还有更多的进阶用法等待你来发掘。

> [你可以通过点击这里来查看使用缓存或者不缓存使用 object visitor 的基准测试](/800-benchmark/002-cached-vs-none-cached-object-visitor)
