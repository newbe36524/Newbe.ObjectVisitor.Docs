---
title: 我的第一个 Object Visitor
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

我们使用 IDE 打开刚刚创建的项目，添加一个简单的数据模型类 OrderInfo :

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

1. new 一个 OrderInfo order
2. 使用 StringBuilder 将 order 的所有属性名称和值拼接在一起
3. 输出最后的 string

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

1. 使用 V() 扩展方法来创建一个 Object Visitor
2. 调用 Object Visitor 的 ForEach 方法来注册 Visit 过程的行为
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
