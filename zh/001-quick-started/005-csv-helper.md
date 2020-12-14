---
title: 0x05-综合示例，导出CSV
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

现在，我们来完成一个稍微复杂一点的场景用例。

## 将实体导出为 CSV 文件

为了使下文的示例更加符合生产实际，我们在这里引入一个具体的场景。

我们需要将实体导出为 CSV 文件。

CSV 文件一般包含两个部分。

第一个部分是文件的头部。头部中包含了每一列的列名。

第二个部分是内容部分。内容部分每行都是一条记录。

不论是头部还是内容部分每个属性之间都使用逗号进行分隔。

这里我们使用前文使用到的 `OrderInfo` 进行演示：

```cs
public class OrderInfo
{
    public int OrderId { get; set; }
    public string Buyer { get; set; }
    public decimal TotalPrice { get; set; }
}
```

则导出的 CSV 样例如下：

```csv
OrderId,Buyer,TotalPrice
1,yueluo,99999
2,newbe36524,36524
3,traceless,123456
```

## 分析实现思路

这个业务场景的实现方式可以非常多样化，此处我们简要将逻辑分为以下部分：

1. 使用 object visitor 访问`OrderInfo`的所有属性
2. 将所有属性传递给 CSV Writer 进行输出

## 实现 CSV 写入器

首先，我们先添加第 2 步所需要的 CSV 写入器：

```cs
public interface ICsvWriter
{
    ICsvWriter WriteHeader(string header);
    ICsvWriter FinishHead();
    ICsvWriter WriteCell(string cell);
    ICsvWriter FinishRow();
}

public class CsvWriter : ICsvWriter
{
    public string Separator { get; set; } = ",";
    private readonly TextWriter _writer;

    public CsvWriter(
        TextWriter writer)
    {
        _writer = writer;
    }

    private bool _firstHead = true;

    public ICsvWriter WriteHeader(string header)
    {
        if (_firstHead)
        {
            _firstHead = false;
        }
        else
        {
            _writer.Write(Separator);
        }

        _writer.Write(header);

        return this;
    }

    public ICsvWriter FinishHead()
    {
        _writer.WriteLine();
        return this;
    }

    private bool _firstCell = true;

    public ICsvWriter WriteCell(string cell)
    {
        if (_firstCell)
        {
            _firstCell = false;
        }
        else
        {
            _writer.Write(Separator);
        }

        _writer.Write(cell);

        return this;
    }

    public ICsvWriter FinishRow()
    {
        _firstCell = true;
        _writer.WriteLine();

        return this;
    }
}
```

有了这样一个基础的`CsvWriter`,我们便可以首先来生成一个样例中的表格：

```cs
var sb = new StringBuilder();
var csvWriter = new CsvWriter(new StringWriter(sb));
csvWriter.WriteHeader("OrderId")
    .WriteHeader("Buyer")
    .WriteHeader("TotalPrice")
    .FinishHead()
    .WriteCell("1")
    .WriteCell("yueluo")
    .WriteCell("99999")
    .FinishRow()
    .WriteCell("2")
    .WriteCell("newbe36524")
    .WriteCell("36524")
    .FinishRow()
    .WriteCell("3")
    .WriteCell("traceless")
    .WriteCell("123456")
    .FinishRow();
Console.WriteLine(sb.ToString());
```

## 输出表头

现在，我们使用第一个 object visitor 来调用上文的 CsvWriter 来输出表头：

```cs
var sb = new StringBuilder();
var csvWriter = new CsvWriter(new StringWriter(sb));

default(OrderInfo)
    .V()
    .WithExtendObject<OrderInfo, CsvWriter>()
    .ForEach((name, value, w) => w.WriteHeader(name))
    .Run(new OrderInfo(), csvWriter);
csvWriter.FinishHead();

Console.WriteLine(sb.ToString());
```

这样我就会得到如下的结果：

```csv
OrderId,Buyer,TotalPrice
```

## 输出表行

现在，我们在增加一个 object visitor 来输出表的每行内容：

```cs
var rowWriter = default(OrderInfo)
    .V()
    .WithExtendObject<OrderInfo, CsvWriter>()
    .ForEach((name, value, w) => w.WriteCell(value != null ? value.ToString() : ""))
    .Cache();

var orders = new List<OrderInfo>
{
    new OrderInfo
    {
        OrderId = 1,
        Buyer = "yueluo",
        TotalPrice = 99999M
    },
    new OrderInfo
    {
        OrderId = 2,
        Buyer = "newbe36524",
        TotalPrice = 36524M
    },
    new OrderInfo
    {
        OrderId = 3,
        Buyer = "traceless",
        TotalPrice = 123456M
    }
};
foreach (var order in orders)
{
    rowWriter.Run(order, csvWriter);
    csvWriter.FinishRow();
}

Console.WriteLine(sb.ToString());
```

这样就会得到如下的内容：

```csv
1,yueluo,99999
2,newbe36524,36524
3,traceless,123456
```

这正是我们期望的表中的行数据。

## 创建 CsvExtensions

现在，我们将以上的表头和表行的相关逻辑进行整合，将他们全部都添加到一个 CsvExtensions 的类型中。并且增加对于`IEnumerable<T>`的扩展方法，这样在进行调用时就会更加简单。这将仿照先前`FormatToStringExtensions`中的做法。

```cs
public static class CsvExtensions
{
    public static string ToCsv<T>(this IEnumerable<T> items)
        where T : new()
    {
        var re = CsvFilerHelper<T>.Instance.ToCsv(items);
        return re;
    }

    private static class CsvFilerHelper<T>
        where T : new()
    {
        internal static readonly ICsvHelper Instance = new CsvHelper();

        public interface ICsvHelper
        {
            string ToCsv(IEnumerable<T> items);
        }

        private class CsvHelper : ICsvHelper
        {
            private readonly ICachedObjectVisitor<T, CsvWriter> _headerWriter;
            private readonly ICachedObjectVisitor<T, CsvWriter> _bodyWriter;

            public CsvHelper()
            {
                _headerWriter = default(T)
                    .V()
                    .WithExtendObject<T, CsvWriter>()
                    .ForEach((name, value, w) => w.WriteHeader(name))
                    .Cache();

                _bodyWriter = default(T)
                    .V()
                    .WithExtendObject<T, CsvWriter>()
                    .ForEach((name, value, w) => w.WriteCell(value != null ? value.ToString() : ""))
                    .Cache();
            }

            public string ToCsv(IEnumerable<T> items)
            {
                var sb = new StringBuilder();
                var csvWriter = new CsvWriter(new StringWriter(sb));
                _headerWriter.Run(new T(), csvWriter);
                csvWriter.FinishHead();
                foreach (var item in items)
                {
                    _bodyWriter.Run(item, csvWriter);
                    csvWriter.FinishRow();
                }

                var re = sb.ToString();
                return re;
            }
        }
    }
}
```

与先前的`FormatToStringExtensions`一样，此处采用的是泛型静态类配合扩展方法的形式来创建帮助方法。不同的是，在这个示例中存在两个 object visitor。因此多考虑抽象了一个`ICsvHelper`和实现类来实现复杂逻辑的聚合。

## 输出时跳过特定的列

我们希望在输出 CSV 的时候跳过一些特定的列，这就需要对属性进行过滤。

我们增加一个新的 Attribute:

```cs
public class IgnoreAttribute : Attribute
{
}
```

然后将这个 Attribute 标记在不希望输出的属性上：

```cs
public class OrderInfo
{
    public int OrderId { get; set; }
    [Ignore]
    public string Buyer { get; set; }
    public decimal TotalPrice { get; set; }
}
```

然后，我们只要在 object visitor 中忽略这些被标记为 Ingore 的属性即可。

其中核心的修改如下：

```cs
static bool Filter(PropertyInfo p) => p.GetCustomAttribute<IgnoreAttribute>() == null;
_headerWriter = default(T)
    .V()
    .WithExtendObject<T, CsvWriter>()
    .FilterProperty(Filter)
    .ForEach((name, value, w) => w.WriteHeader(name))
    .Cache();

_bodyWriter = default(T)
    .V()
    .WithExtendObject<T, CsvWriter>()
    .FilterProperty(Filter)
    .ForEach((name, value, w) => w.WriteCell(value != null ? value.ToString() : ""))
    .Cache();
```

这样我们在输出 CSV 时也就不存在这一列了：

```csv
OrderId,TotalPrice
1,99999
2,36524
3,123456
```

## 总结

我们通过一个简单的生产实例来理解 object visitor 的用法。实际生产问题会比这个更加复杂。开发者可以在生产实际中进行尝试，强化理解。
