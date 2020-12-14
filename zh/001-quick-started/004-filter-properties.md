---
title: 0x04-过滤属性
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

我们已经掌握了 `ForEach` 的完整用法，现在我们来进一步了解一下如何按照需求来“过滤属性”。

所谓“过滤属性”，是指在创建 object visitor 过程中跳过那些不满足条件的属性。

`ForEach` 的重载一共分为泛型和非泛型两个版本。这两者的过滤方式存在一定区别。

## ForEach 非泛型方式属性过滤

首先是最简单的重载形式：

```cs
ForEach(Expression<Action<string, object>> foreachAction)
```

有两种方式对其进行过滤：

- 调用扩展方法 `ForEach(Expression<Action<string, object>> foreachAction, Func<PropertyInfo, bool>? propertyInfoFilter)`
- 使用 FluentAPI `FilterProperty(Func<PropertyInfo, bool>? propertyInfoFilter).ForEach(Expression<Action<string, object>> foreachAction)`

这两种方式得到的结果完全一样。我们以 FluentAPI 的方式举例说明：

所有的属性都将被访问。这也是未指定的 filter 时的默认行为。

```cs
FilterProperty(p => true).ForEach((name, value) => _ )
```

`Name` 为 `"P_"` 开头的属性才会被访问。

```cs
FilterProperty(p => p.Name.StartWith("P_")).ForEach((name, value) => _ )
```

只有`string`类型的属性才会被访问。

```cs
FilterProperty(p => p.PropertyType == typeof(string)).ForEach((name, value) => _ )
```

属性被标记为 RequiredAttribute 才会被访问

```cs
FilterProperty(p => p.GetCustomAttribute<RequiredAttribute>() != null ).ForEach((name, value) => _ )
```

另外还有一些小的注意点：

1. 同一个`ForEach`下`FilterProperty`虽然可以被多次调用，但是只会保留最后一个效果。
2. 如果有多个`ForEach`，我们将以使用扩展方法形式调用。这样你不会容易使自己混乱。

## ForEach 泛型方式属性过滤

泛型的最简重载形式如下：

```cs
ForEach<TValue>(Expression<Action<string, TValue>> foreachAction)
```

与非泛型版本一样，泛型版本一样可以通过扩展方法和 FluentAPI 来指定属性过滤条件。我们以 FluentAPI 的方式举例说明：

所有的`string`属性都将被访问。这也是未指定的 filter 时的默认行为。这将跳过非`string`类型的属性。这两者是完全等效的。

```cs
ForEach<string>((name, value) => _ )

FilterProperty(p => p.PropertyType == typeof(string)).ForEach<string>((name, value) => _ )
```

所有的属性都将被访问，并且在访问时会尝试强制转换为`string`，如果强制转换失败，将会引发异常。注意这和前一个例子不相同的行为。

```cs
FilterProperty(p => true).ForEach<string>((name, value) => _ )
```

因此，和非泛型版本相比，泛型版本最大的区别就是以下：

1. 访问时将会发生强制转换，如果转换失败将会引发异常
2. 默认不写过滤条件时将会按照给定的泛型类型进行过滤。

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
