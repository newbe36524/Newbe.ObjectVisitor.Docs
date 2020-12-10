---
title: Newbe.ObjectVisitor
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

# Newbe.ObjectVisitor

- [简体中文](/zh/home)
- [English](/en/home)

![banner.svg](/icons/banner.svg)

Newbe.ObjectVisitor helps developers access all the properties of a normal class in the simplest and most efficient way.This enables：validation, mapping, collection, and more.

For example, there is such a simple class in your code.

```cs
var order = new OrderInfo();
```

If you want to print out all the properties and values of this class, you can use reflection to：

```cs
for(var pInfo in typeof(OrderInfo).GetProperties())
{
    Console.Writeline($"{pInfo.Name}: {pInfo.GetValue(order)}");
}
```

If you use this class library, you can achieve the same effect by：

```cs
// invoke  .V
// there is a visitor for OrderInfo
var visitor = order.V();

visitor.ForEach(context=>{
    var name = context.Name;
    var value = context.Value;
    Console.Writeline($"{name}: {value}");
}).Run();

// it can be joined into one line.
order.V().ForEach(c=> Console.Writeline($"{c.Name}: {c.Value}")).Run();

// or shorter
order.FormatToString();
```

[You can click here to get into the "Quick Start" and learn about the full documents](/001-quick-started/001-my-fisrt-object-visitor)

## So why do I have to do this?

- **because it's so much faster!** this class library is implemented using[expression tree](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/), so it has 10 times faster performance than direct reflection.
- **because it's more readable!** this class library you can use chained APIs and naming methods to create a delegate that allows your code to implement the same readable effect as hard-coded.
- **because it's more extendable!** if you use this class library, you have an easy way to access all the properties of a class.So you do a lot of things you want to do, like：create a validator to validate your model, modify properties that might contain sensitive data to avoid output to logs, create an object mapper similar to AutoMapper but have better performance, and so on.

## Contacts

- QQ group: 【Newbe.Claptrap CL4P-TP 610394020 】：<https://jq.qq.com/?_wv=1027&k=Lkhbwj0o>
- Discord：<https://discord.gg/6yd3mK6M>

## Contributing

We still need more friends to get involved in this intense project.

[You can click here for detailed contributions](/900-contribution/001-welcome-contributors)

## Stargazers over time

[![Stargazers over time](https://starchart.cc/newbe36524/Newbe.ObjectVisitor.svg)](https://starchart.cc/newbe36524/Newbe.ObjectVisitor)
