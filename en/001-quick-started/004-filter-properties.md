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

## The filtering properties occur when the object visitor is building

object visitor is created and used separately.The essence is the distinction between the construction and compilation steps of expressions.The filtering property action occurs during the build process, i. e. the compiled code about the filtered property fields will disappear directly.

We combine the examples in the`FormatToStringExtensions`to understand.

Before filtering properties, the delegate generated by object visitor may look like the following code：

```cs
sb.AppendFormat("{0}: {1}{2}", nameof(order.OrderId), order.OrderId, Environment.NewLine);
sb.AppendFormat("{0}: {1}{2}", nameof(order.Buyer), order.Buyer, Environment.NewLine);
sb.AppendFormat("{0}: {1}{2}", nameof(order.TotalPrice), order.TotalPrice, Environment.NewLine);
```

However, if you add a filter that excludes`Buyer`property, the generated code becomes：

```cs
sb.AppendFormat("{0}: {1}{2}", nameof(order.OrderId), order.OrderId, Environment.NewLine);
sb.AppendFormat("{0}: {1}{2}", nameof(order.TotalPrice), order.TotalPrice, Environment.NewLine);
```

Yes, it's the code that disappears directly.Instead of adding`if`judgment.

## Summary

Filtering properties can be used to skip the properties that do not want to participate in the object visitor access process. This is usually required in some specific scenarios.

Developers can use example in [0x05-comprehensive example, exporting CSV](/001-quick-started/005-csv-helper) to understand object visitor.
