---
title: 0x02-Create and cache the Object Visitor
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## It's all about being more efficient

The previous piece, we present through a simple instance how to use Object Visitor to connect all the properties of the `OrderInfo` and output it.

Although the effect has already been achieved, there are some underperforming practices to simplify space, and this section describes the truly complete posture using Newbe.ObjectVisitor.

## Cached object visitor

The core code of the previous section is as follows：

```cs
var sb2 = new StringBuilder();
var visitor = order.V()
    .ForEach((name, value) => sb2.AppendFormat("{0}: {1}{2}", name, value, Environment.NewLine));
visitor.Run();
Console.WriteLine(sb2.ToString());
```

In this code, we create an object visitor directly, then use it directly later, and then the method ends directly.

This is no problem for one-time object visitor.But if this code is on a common hotspot path, it's a waste to create object visitor over and over again.Therefore, we can consider caching object visitor in a static field.

So we got the following code：

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

Here, we made these changes：

1. The object visitor is modified to a static field and is assigned using field initialization.
2. When you initialize object visitor, because the `OrderInfo` object has not yet been determined, use `default(OrderInfo)` to represent the type of target object visitor needs to access.
3. We called the `Cache` method after the `ForEach` to create a cached object visitor.The details of the implementation can be explained here： `Cache` actually compiles the expression into a delegate so that it can be used repeatedly later for best performance.
4. Because it is initialized in a static field, the `StringBuilder` object in `ForEach` must also be a static field, so we also define it in the `_sb` field.
5. With the above modification, we can create formatted strings by cached `_visitor`.

## You also need Extend Object

In the last section, we cached object visitor avoiding the overhead of creating object visitor multiple times.

However, a very serious problem is left：`ForEach` uses static field `_sb` directly for string stitching, which results in multiple calls to `_visitor. Run(order)` will continue to stitch on the original basis.This is clearly unreasonable.

In response to the possibility that an additional object (`StringBuilder`) might be required in addition to the accessed object (`order`) during the object visitor run, we added a feature called Extend Object.

Next, let's show you the example:

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

Here, we made these changes：

1. Using `WithExtendObject` extension method, it is specified that our object visitor needs to pass in a `StringBuilder` type as Extend Object at runtime.
2. Modified `ForEach` to receive externally `StringBuilder`and call its stitching method.
3. Modified `_visitor.Run` method.Because, at this point it is not just necessary to pass `order`, it is also necessary to pass a `StringBuilder` as an Extends Object.
4. Run this example and you'll find that you've output the same information twice, and that's exactly what we want.

With the Extend Object we are able to accept some additional parameters in the `ForEach` in order to implement some logic.

## We can still optimize it for a little bit more.

We can use generic static classes to cache our object visitor so that we can implement all types of property stitching：

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

Here, we made these changes：

1. We've added a `FormatToStringExtensions` static class to add an extension method `FormatToString`so that all type objects have the `FormatToString` method.
2. Added a `FormatStringVisitor<T>` generic static class that creates and saves an object visitor instance to the `Instance` property so that each type T has its own separate object visitor.
3. Calling the `FormatToString` method where you need to format, you can complete the same logic that you previously needed to achieve.

This is the ultimate full-body solution.In fact,`FormatToString` method is now built into the Newbe.ObjectVisitor package, and its implementation is as described above：

[Click here to see the source code of FormatToStringExtensions](https://github.com/newbe36524/Newbe.ObjectVisitor/blob/main/src/Newbe.ObjectVisitor/Newbe.ObjectVisitor/Util/FormatToStringExtensions.cs)

## Summary

With this piece, we get the best paradigm with the object visitor through a series of optimisation.

The adoption of the best paradigm has been able to be applied in most of the scenarios.

Of course, there are more advanced usage waiting for you to discover.

> [You can check the use of the cache or not cache the benchmark test using the object visitor by clicking here](/800-benchmark/002-cached-vs-none-cached-object-visitor)
