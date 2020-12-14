---
title: 0x05-Integrated Example, Export CSV
description:
published: true
date: 2020-11-26T14:51:47.581Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

Now let's complete a slightly more complex scenario use case.

## Export entities as CSV files

In order for the examples below to be more in line with production realities, we introduce a specific scenario here.

We need to export the entity as a CSV file.

CSV files typically consist of two parts.

The first part is the head of the file.The header contains the column name for each column.

The second part is the content part.Each line in the content section is a record.

Each property, whether header or content section, is separated by a comma.

Here we use the previously used `OrderInfo` for presentation:

```cs
public class OrderInfo
{
    public int OrderId { get; set; }
    public string Buyer { get; set; }
    public decimal TotalPrice { get; set; }
}
```

The CSV-like cases that are exported are as follows：

```csv
OrderId,Buyer,TotalPrice
1,yueluo,99999
2,newbe36524,36524
3,traceless,123456
```

## Analyze the implementation ideas

The way in which this business scenario is realized can be very diversified, here we briefly divide the logic into the following sections：

1. 使用 object visitor 访问`OrderInfo`的所有属性
2. 将所有属性传递给 CSV Writer 进行输出

## Implement the CSV writer

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

## The output header

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

## The output table row

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

## Create CsvExtensions

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

## Skip a specific column when output

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

## Summary

我们通过一个简单的生产实例来理解 object visitor 的用法。实际生产问题会比这个更加复杂。开发者可以在生产实际中进行尝试，强化理解。
