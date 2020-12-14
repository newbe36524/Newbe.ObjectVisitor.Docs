---
title: 0x01 - My first Object Visitor
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## Preparation

In order to test smoothly, you need to make sure that the following prerequisites are installed locally：

- dotnet 2.1 or the SDK of the above version, we are more recommended to install the dotnet 5 SDK directly.Download address：<https://dotnet.microsoft.com/download>
- Install a .net IDE at your advantage.This demo process will use Visual Studio Code as the basis for the demo IDE.You can also choose Rider or Visual Studio to do it.

## Create a test project

We need a test project to demonstrate how to create your first Object Visitor.

First, make sure that dotnet 2.1 or above versions of sdk are installed correctly locally, and you can run the following commands to confirm the currently installed version:

```bash
dotnet --info
```

Then, find a folder you like and run the following commands to create a test console program for demonstration:

```bash
dotnet new console
```

Finally, continue with the following commands to install the latest Newbe.ObjectVisitor package:

```bash
dotnet add package newbe.objectvisitor
```

In this way, the project used for testing is created and completed.

## Create a simple data model

Let's use the IDE to open the project we just created and add a simple data model `OrderInfo` :

```cs
public class OrderInfo
{
    public int OrderId { get; set; }
    public string Buyer { get; set; }
    public decimal TotalPrice { get; set; }
}
```

## Realize the logic of a splicing all the properties

We add the following code in Program.cs to complete these logic：

1. `new` a `OrderInfo` `order`
2. Stitch together all the property names and values `order` with the `StringBuilder`
3. Output the `string`

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

We use the following commands to run the written code:

```bash
dotnet run
```

It's going to get the following results：.

```bash
OrderId: 1
Buyer: yueluo
TotalPrice: 62359.1478
```

## Use Object Visitor to implement the logic above again

Let's do the same with Newbe.ObjectVisitor to implement the logic：

1. Create Object Visitor using `V()` extension method
2. Call Object Visitor's `ForEach` method to register behavior of the Visit process
3. Run the created Object Visitor

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

Running this code, you're going to get the same results as the previous section.

## So what's the benefit of doing that?

First, the use of Object Visitor can dynamically adapt to changes in model classes, the benefit of which is very obvious.

As properties of `OrderInfo` change in the file, the "stitched part" code can be dynamically adapted without change.

Also, there are some benefits that are not embodied in this example and will be introduced in subsequent documents：

1. It operates efficiently.According to already existing benchmark tests, their performance is very similar with direct hard coding.
2. There are a number of ways to filter properties that need access, such as: based `Attribute` filtering.
3. Having this way can be followed by an easy extension of other functions based on the object properties, such as the property verification (FluentValidation), the mapping of objects (AutoMapper) and the comparison of objects (Comparer).

## So what is the difference between this and direct use of reflection?

It is also possible to use reflection to achieve the above effects, but Object Visitor is implemented in a performance-based manner:

1. Based on existing benchmarks, Object Visitor is implemented based on expression trees and is much more efficient than using reflection-related read and write methods directly.
2. Object Visitor provides generic-based methods, and in some specific scenarios the overhead of unboxing can be completely avoided.

> [You can see benchmarks for using reflection and object visitor by clicking here](/800-benchmark/001-object-visitor-vs-relfection-vs-directly)

## Summary

Here, you've got a first look at what Newbe.ObjectVisitor is and its basic usage.

However, this is just demo code and the content is very limited, so you can continue reading the next section to learn more about the features of tension stimulation.

You can also join the following groups to discuss with us:

- QQ group: 【Newbe.Claptrap CL4P-TP 610394020 】：<https://jq.qq.com/?_wv=1027&k=Lkhbwj0o>
- Discord：<https://discord.gg/6yd3mK6M>
