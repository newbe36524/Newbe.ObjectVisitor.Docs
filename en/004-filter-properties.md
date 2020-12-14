---
title: 0x04-Filter Properties
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

We have mastered the complete usage of `ForEach`, and now we come to learn more about how to "filter properties" according to the requirements.

The so-called "filtering property" refers to the skipping of those properties that do not meet the conditions during the creation of the object visitor.

`ForEach` are divided into generic and non-generic versions.There is a certain difference between the way the two are filtered.

## ForEach non-generic property filtering

The first is the simplest form of overload：

```cs
ForEach(Expression<Action<string, object>> foreachAction)
```

There are two ways to filter it：

- Call the extension `ForEach(Expression<Action<string, object>> foreachAction, Func<PropertyInfo, bool>? propertyInfoFilter)`
- Using FluentAPI `FilterProperty(Func<PropertyInfo, bool>? propertyInfoFilter).ForEach(Expression<Action<string, object>> foreachAction)`

The results in both ways are exactly the same.Let's give an example of this in the form of fluentAPI：

All properties will be accessed.This is also the default behavior when filtering is not specified.

```cs
FilterProperty(p => true).ForEach((name, value) => _ )
```

Property `Name` starting with `"P_"` will only be accessed.

```cs
FilterProperty(p => p.Name.StartWith("P_")).ForEach((name, value) => _ )
```

Only`string`types of properties will be accessed.

```cs
FilterProperty(p => p.PropertyType == typeof(string)).ForEach((name, value) => _ )
```

The property is marked as RequiredAttribute to be accessed

```cs
FilterProperty(p => p.GetCustomAttribute<RequiredAttribute>() != null ).ForEach((name, value) => _ )
```

There are also a few small points:

1. `FilterProperty` next to the same`ForEach` although can be called multiple times, only the last effect will be preserved.
2. If there are multiple`ForEach`, we recommend to call it in the form of an extended method.That way you won't easily mess yourself up.

## ForEach generic property filtering

The simplest overload form of generics is:

```cs
ForEach<TValue>(Expression<Action<string, TValue>> foreachAction)
```

As with non-generic versions, generic versions can specify property filters by extending methods and FluentAPI.Let's give an example of this in the form of fluentAPI：

All`string`properties will be accessed.This is also the default behavior when filtering is not specified.This skips the non-`string`type of property.The two are perfectly equivalent to each other.

```cs
ForEach<string>((name, value) => _ )

FilterProperty(p => p.PropertyType == typeof(string)).ForEach<string>((name, value) => _ )
```

All the properties will be accessed, and will try to force conversion to`string`at the time of the visit, which will cause an exception if the forced conversion fails.Note that this is not the same behavior as in the previous example.

```cs
FilterProperty(p => true).ForEach<string>((name, value) => _ )
```

Therefore, the biggest difference between generic versions and non-generic versions is the following：

1. There will be a forced conversion at the time of the visit, if the conversion fails will cause an exception
2. When the filter is not written by default, it is filtered by a given generic type.

## 过滤属性发生在构建 object visitor 时

object visitor 的创建和使用是分开的。其本质就是表达式的构建和编译步骤的区分。而过滤属性这个操作就是发生在构建的过程中，即编译出来的代码关于被过滤属性的字段会直接消失。

我们结合`FormatToStringExtensions`中的示例来理解。

在没有过滤属性之前，object visitor 生成的委托可能形如如下代码：

```cs
sb.AppendFormat("{0}: {1}{2}", nameof(order.OrderId), order.OrderId, Environment.NewLine);
sb.AppendFormat("{0}: {1}{2}", nameof(order.Buyer), order.Buyer, Environment.NewLine);
sb.AppendFormat("{0}: {1}{2}", nameof(order.TotalPrice), order.TotalPrice, Environment.NewLine);
```

但是如果，增加了过滤条件，将`Buyer`属性排除在外，那么生成的代码就会变为：

```cs
sb.AppendFormat("{0}: {1}{2}", nameof(order.OrderId), order.OrderId, Environment.NewLine);
sb.AppendFormat("{0}: {1}{2}", nameof(order.TotalPrice), order.TotalPrice, Environment.NewLine);
```

是的，是这段代码直接消失。而不是增加了`if`判断。

## 总结

过滤属性可以用于跳过那些不想要参与 object visitor 访问过程的属性。 这通常在一些特定的场景下需要。

开发者可以通过[0x05-综合示例，导出 CSV](/001-quick-started/005-csv-helper)来了解具体的应用场景。
