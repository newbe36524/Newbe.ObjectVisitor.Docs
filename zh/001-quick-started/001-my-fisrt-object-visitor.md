---
title: 0x01-我的第一个 Object Visitor
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## 预演准备

为了顺利的进行测试，你需要确保本地已经安装了以下这些必备的软件：

- dotnet 2.1 或者以上版本的 SDK，我们更建议直接安装 dotnet 5 SDK。下载地址：<https://dotnet.microsoft.com/download>
- 安装一个趁手的 .net IDE。本演示过程将会使用 Visual Studio Code 作为基础演示 IDE。你也可以选择 Rider 或者 Visual Studio 来完成。

## 创建测试项目

我们需要一个测试项目来演示如何创建一个属于你的第一个 Object Visitor。

首先，确保本地已经正确安装了 dotnet 2.1 或者以上版本的 sdk，你可以运行以下命令来确认当前安装的版本:

```bash
dotnet --info
```

然后，随便找一个你喜欢的文件夹，运行以下命令来创建用于演示的测试控制台程序:

```bash
dotnet new console
```

最后，继续运行以下命令来安装最新的 Newbe.ObjectVisitor 工具包:

```bash
dotnet add package newbe.objectvisitor
```

这样，用于测试的项目就创建完成了。

## 创建一个简单的数据模型

我们使用 IDE 打开刚刚创建的项目，添加一个简单的数据模型类 `OrderInfo` :

```cs
public class OrderInfo
{
    public int OrderId { get; set; }
    public string Buyer { get; set; }
    public decimal TotalPrice { get; set; }
}
```

## 实现一个拼接所有属性的逻辑

我们在 Program.cs 中添加以下代码来完成这些逻辑：

1. `new` 一个 `OrderInfo` `order`
2. 使用 `StringBuilder` 将 `order` 的所有属性名称和值拼接在一起
3. 输出最后的 `string`

```cs
using System;
using System.Text;

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
            var sb = new StringBuilder();
            sb.AppendFormat("{0}: {1}{2}", nameof(order.OrderId), order.OrderId, Environment.NewLine);
            sb.AppendFormat("{0}: {1}{2}", nameof(order.Buyer), order.Buyer, Environment.NewLine);
            sb.AppendFormat("{0}: {1}{2}", nameof(order.TotalPrice), order.TotalPrice, Environment.NewLine);
            Console.WriteLine(sb.ToString());
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

我们使用以下命令来运行写好的逻辑：

```bash
dotnet run
```

就会得到以下这样的结果：

```bash
OrderId: 1
Buyer: yueluo
TotalPrice: 62359.1478
```

## 使用 Object Visitor 再次实现上面的逻辑

我们通过 Newbe.ObjectVisitor 来一样实现上面的逻辑：

1. 使用 `V()` 扩展方法来创建一个 Object Visitor
2. 调用 Object Visitor 的 `ForEach` 方法来注册 Visit 过程的行为
3. 运行创建好的 Object Visitor

```cs
using System;
using System.Text;

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

            var sb2 = new StringBuilder();
            var visitor = order.V()
                .ForEach((name, value) => sb2.AppendFormat("{0}: {1}{2}", name, value, Environment.NewLine));
            visitor.Run();
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

运行这个代码，你将会得到和上一节相同的结果。

## 那这么做究竟带来了什么好处？

首先，使用 Object Visitor 可以动态的适应模型类的变化，这点好处非常明显。

当 `OrderInfo` 中的属性增加时，“拼接部分”的代码可以不用变化，实现动态的适配。

另外，还有一些好处是本示例没有体现的，将会在后续的文档中进行介绍：

1. 它的运行效率很高。根据已有的基准测试，其性能表征和直接硬编码差距很小。
2. 可以使用丰富的方式来对需要访问的属性进行多种方式过滤，例如：基于 `Attribute` 的过滤。
3. 有了这种方式之后可以很轻松的扩展出基于对象属性的其他功能，例如：对象的属性验证（FluentValidation），对象的映射（AutoMapper）和对象的比较（Comparer）。

## 那这和直接使用反射有什么区别？

使用反射来实现以上的效果也是可以的，但相较来说，Object Visitor 的实现方式在性能方面根据优势：

1. 根据已有的基准测试，Object Visitor 基于表达式树实现，其运行效率要比直接使用反射相关的读写方法高出许多。
2. Object Visitor 提供了基于泛型，在一些特定的场景可以完全避免装箱拆箱所带来的开销。

> [你可以通过点击这里来查看使用反射和使用 object visitor 的基准测试](/800-benchmark/001-object-visitor-vs-relfection-vs-directly)

## 本篇小结

到这里，你已经初步了解了什么是 Newbe.ObjectVisitor，以及其基本的用法。

不过，这只是演示代码，展现的内容非常有限，因此，你还可以继续阅读下篇来进一步了解更多紧张刺激的特性。

你也可以加入以下这些群组来和我们一起讨论：

- QQ 群: 【Newbe.Claptrap CL4P-TP 610394020 】：<https://jq.qq.com/?_wv=1027&k=Lkhbwj0o>
- Discord：<https://discord.gg/6yd3mK6M>
