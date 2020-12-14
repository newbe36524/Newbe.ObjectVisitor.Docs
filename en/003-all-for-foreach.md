---
title: 0x03-ForEach Full View
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

We've learned about the most basic parts that make up an object visitor and the best performance practices.This article we come to introduce more about the strange operation of the `ForEach` method.

## Overload of ForEach

The following major overloaded forms of ForEach：

```cs
ForEach(Expression<Action<IObjectVisitorContext<T, object>>> foreachAction)
ForEach(Expression<Action<string, object>> foreachAction)

ForEach<TValue>(Expression<Action<IObjectVisitorContext<T, TValue>>> foreachAction)
ForEach<TValue>(Expression<Action<string, TValue>> foreachAction)
```

These four overloads are obtained by crossing "generic" and "using Context" respectively.

### Overload with Context ForEach

Let's start by explaining the difference between using Context overloading.

In this normal overload below, you can only get the name and value properties.

```cs
ForEach(Expression<Action<string, object>> foreachAction)
```

This is also the most basic form of overloading, allowing you to read the names and values of all properties.

But you may also need more contextual information, so you've introduced an overloaded form that you can access using Context：

```cs
ForEach(Expression<Action<IObjectVisitorContext<T, object>>> foreachAction)
```

This brings some advantages：

- You have access to more information than Name and Value, and currently includes at least：SourceObject, which represents the original access object, and PropertyInfo, which represents the property information
- Context is not completely read-only.Where the Value property allows assignments.Therefore, you can assign values to modify property values in the original object.This cannot be done by parameter passing (ref is not used).
- The method signature does not change because there is only one parameter.It is not necessary to modify the method signature because of the added properties.

But it's going to bring some disadvantage:

- Method signatures are longer
- A Context object is created when run, which brings a very slight performance overhead

Therefore, if the advantages above outweigh the losses in your scenario, you might also consider using Context.

### Overload with generic ForEach

Using ForEach's generic overload can make the resulting Value a property value of an explicit type, not an object type.

By doing so, you can get the following benefits：

- For value types you can avoid boxing and unboxing consumption
- You can explicitly use pre-known types

Of course, you've lost some：

- You can't handle all properties in general, especially if the property types are different in one type

When using generic overloading, it is also important to filter the types that need access.After all, not all types in one object can fit into the same generic type.We'll explain this in the next section.

Context also has a generic version of the overload.Its role is to combine Context and generics.

## Add extension data

Similar to regular overloading, if an object visitor is specified to pass extended data.Then its overload will also change, allowing the incoming extended data to be processed：

```cs
ForEach(Expression<Action<IObjectVisitorContext<T, TExtend, object>>> foreachAction)
ForEach(Expression<Action<string, object, TExtend>> foreachAction)

ForEach<TValue>(Expression<Action<IObjectVisitorContext<T, TExtend, TValue>>> foreachAction)
ForEach<TValue>(Expression<Action<string, TValue, TExtend>> foreachAction)
```

This point developers should be very familiar with, since the object visitor with extended data is used in the "formatted objects" example used in our previous article.

Similar to regular overloads, the above overloads also have context and generic forms, and the functionality is similar.There is just not much to say about it here.

## ForEach is called multiple times

ForEach is allowed to be called multiple times to register in an object visitor.

It is interesting to note that these registers share the same original object with each other, so that the modification of the previous ForEach will have an impact on the latter.

Here's an example of us adding strings from order to list1.However, after the first addition, its property value is modified to empty.So list2 can't get a value.

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

This code will output the following results：

```cs
1
0
```
