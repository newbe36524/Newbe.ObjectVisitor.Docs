---
title: Modify the property
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## Modify the property

The physical machine used by the following benchmark tests:

```ini

BenchmarkDotNet=v0.12.1, OS=Windows 10.0.19041.572 (2004/?/20H1)
Intel Xeon CPU E5-2678 v3 2.50GHz, 1 CPU, 24 logical and 12 physical cores
.NET Core SDK=5.0.100
  [Host]       : .NET Core 2.1.23 (CoreCLR 4.6.29321.03, CoreFX 4.6.29321.01), X64 RyuJIT
  net461       : .NET Framework 4.8 (4.8.4250.0), X64 RyuJIT
  net48        : .NET Framework 4.8 (4.8.4250.0), X64 RyuJIT
  netcoreapp21 : .NET Core 2.1.23 (CoreCLR 4.6.29321.03, CoreFX 4.6.29321.01), X64 RyuJIT
  netcoreapp31 : .NET Core 3.1.9 (CoreCLR 4.700.20.47201, CoreFX 4.700.20.47203), X64 RyuJIT
  netcoreapp5  : .NET Core 5.0.0 (CoreCLR 5.0.20.47505, CoreFX 5.0.20.47505), X64 RyuJIT


```

Now you may need to replace the Password property value in one object with a '\*\*\*'.We can adopt the following programme to achieve：

| Method       | Description                                               |
| ------------ | --------------------------------------------------------- |
| Directly     | Make a modification directly with an assignment statement |
| UsingVisitor | Use the cached ObjectVisitor to make modifications        |

Chart:

![newbe.objectvisitor.benchmarktest.changepasswordtest-barplot.png](/benchmark/newbe.objectvisitor.benchmarktest.changepasswordtest-barplot.png)

Data:

| Method       | Job          | Runtime       |       Mean |    Error |   StdDev | Ratio | RatioSD | Rank |
| ------------ | ------------ | ------------- | ----------:| --------:| --------:| -----:| -------:| ----:|
| Directly     | net461       | .NET 4.6.1    | 1,205.4 ns |  8.90 ns |  7.44 ns |  1.00 |    0.00 |    1 |
| UsingVisitor | net461       | .NET 4.6.1    | 3,807.0 ns | 68.28 ns | 63.87 ns |  3.15 |    0.04 |    2 |
|              |              |               |            |          |          |       |         |      |
| Directly     | net48        | .NET 4.8      | 1,205.9 ns |  5.74 ns |  5.08 ns |  1.00 |    0.00 |    1 |
| UsingVisitor | net48        | .NET 4.8      | 3,743.3 ns | 18.51 ns | 15.46 ns |  3.11 |    0.02 |    2 |
|              |              |               |            |          |          |       |         |      |
| Directly     | netcoreapp21 | .NET Core 2.1 |   999.3 ns |  7.28 ns |  6.08 ns |  1.00 |    0.00 |    1 |
| UsingVisitor | netcoreapp21 | .NET Core 2.1 | 2,882.4 ns |  9.58 ns |  8.96 ns |  2.89 |    0.02 |    2 |
|              |              |               |            |          |          |       |         |      |
| Directly     | netcoreapp31 | .NET Core 3.1 |   807.9 ns |  3.46 ns |  3.07 ns |  1.00 |    0.00 |    1 |
| UsingVisitor | netcoreapp31 | .NET Core 3.1 | 2,614.1 ns | 13.79 ns | 12.90 ns |  3.24 |    0.02 |    2 |
|              |              |               |            |          |          |       |         |      |
| Directly     | netcoreapp5  | .NET Core 5.0 |   533.8 ns |  1.72 ns |  1.44 ns |  1.00 |    0.00 |    1 |
| UsingVisitor | netcoreapp5  | .NET Core 5.0 | 1,398.0 ns |  9.24 ns |  8.19 ns |  2.62 |    0.02 |    2 |

Conclusions:

1. The use of the visitor consumes an extra 1-3 microseconds (a millionth of a second).So if you feel that this time can be accepted, then even though it is used.
