---
title: 修改属性
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## 修改属性

以下基准测试所使用的物理机配置：

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

现在，你可能需要将一个对象中的 Password 属性值替换为'\*\*\*'。我们可以采用以下方案实现：

| 方法           | 描述                       |
| ------------ | ------------------------ |
| Directly     | 直接使用赋值语句进行修改             |
| UsingVisitor | 使用缓存的 ObjectVisitor 进行修改 |

图表:

![newbe.objectvisitor.benchmarktest.changepasswordtest-barplot.png](/benchmark/newbe.objectvisitor.benchmarktest.changepasswordtest-barplot.png)

数据:

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

结论:

1. 使用 visitor 会额外消耗 1-3 微秒（百万分之一秒）。所以如果你觉得这点时间可以接受，那就尽管使用。
