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

1. Access to all the properties of`OrderInfo`with the object visitor
2. Pass all the properties to the CSV Writer for output

## Implement the CSV writer

First, let's add the CSV writer required for step 2：

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

With such a basic`CsvWriter`, we can first generate a table in a sample：

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

Now, let's use the first object visitor to call CsvWriter above to output the header：

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

That way I'll get the results as follows：.

```csv
OrderId,Buyer,TotalPrice
```

## The output table row

Now, we're adding an object visitor to output each row of the table：

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

This will get the following content：

```csv
1,yueluo,99999
2,newbe36524,36524
3,traceless,123456
```

This is exactly the row data in the table we expect.

## Create CsvExtensions

Now, we've consolidated the logic associated with the headers and table rows above, adding them all to a CsvExtensions class.And increasing extension method for`IEnumerable<T>`, so that it will be simpler when making calls.This follows the`FormatToStringExtensions`previously.

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

As with the previous`FormatToStringExtensions`, the help method is created here in the form of a generic static class mate extension method.The difference is that there are two object visitors in this example.So consider abstracting an`ICsvHelper`and implementing classes to achieve the aggregation of complex logic.

## Skip a specific column when output

We want to skip some specific columns when we output CSV, which requires filtering the properties.

Let's add a new Attribute:

```cs
public class IgnoreAttribute : Attribute
{
}
```

Then mark this Attribute on properties that you don't want to：

```cs
public class OrderInfo
{
    public int OrderId { get; set; }
    [Ignore]
    public string Buyer { get; set; }
    public decimal TotalPrice { get; set; }
}
```

Then we just ignore these properties that are marked as Ingore in object visitor.

The changes to the core are as follows：

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

So we don't have this column when we output CSV：

```csv
OrderId,TotalPrice
1,99999
2,36524
3,123456
```

## Summary

We use a simple production example to understand the use of object visitor.The actual production problem will be more complicated than this.Developers can experiment in production practices to enhance understanding.
