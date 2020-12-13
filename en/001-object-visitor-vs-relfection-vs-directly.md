---
title: Newbe.ObjectVisitor vs Reflection vs Directly
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

## Newbe.ObjectVisitor vs Reflection vs Directly

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

### Newbe.ObjectVisitor vs Reflection vs Directly

我们将会把属性的名称和值拼接为一个字符串，采用以下这些方案：

| 方法           | 描述                                                                   |
| ------------ | -------------------------------------------------------------------- |
| Directly     | 使用 StringBuilder 硬编码进行拼接                                             |
| CacheVisitor | 使用 Newbe.ObjectVisitor 创建一个 ObjectVisitor 并缓存它，然后使用缓存后的 visitor 进行拼接 |
| QuickStyle   | 使用 Newbe.ObjectVisitor 中内置写好的方法                                      |

图表:

![newbe.objectvisitor.benchmarktest.formatstringtest-barplot.png](/benchmark/newbe.objectvisitor.benchmarktest.formatstringtest-barplot.png)

数据:

| Method       | Job          | Runtime       |     Mean |    Error |   StdDev | Ratio | RatioSD | Rank |
| ------------ | ------------ | ------------- | --------:| --------:| --------:| -----:| -------:| ----:|
| Directly     | net461       | .NET 4.6.1    | 754.3 ns |  8.49 ns |  7.94 ns |  1.00 |    0.00 |    1 |
| QuickStyle   | net461       | .NET 4.6.1    | 818.3 ns | 16.29 ns | 24.87 ns |  1.10 |    0.04 |    3 |
| CacheVisitor | net461       | .NET 4.6.1    | 791.4 ns | 11.62 ns | 10.30 ns |  1.05 |    0.01 |    2 |
|              |              |               |          |          |          |       |         |      |
| Directly     | net48        | .NET 4.8      | 738.5 ns |  7.37 ns |  6.90 ns |  1.00 |    0.00 |    1 |
| QuickStyle   | net48        | .NET 4.8      | 799.0 ns | 10.63 ns |  9.42 ns |  1.08 |    0.01 |    2 |
| CacheVisitor | net48        | .NET 4.8      | 788.0 ns |  8.27 ns |  6.91 ns |  1.07 |    0.02 |    2 |
|              |              |               |          |          |          |       |         |      |
| Directly     | netcoreapp21 | .NET Core 2.1 | 768.6 ns |  9.63 ns |  9.01 ns |  1.00 |    0.00 |    1 |
| QuickStyle   | netcoreapp21 | .NET Core 2.1 | 787.6 ns |  6.11 ns |  5.42 ns |  1.02 |    0.02 |    2 |
| CacheVisitor | netcoreapp21 | .NET Core 2.1 | 768.6 ns |  5.30 ns |  4.96 ns |  1.00 |    0.01 |    1 |
|              |              |               |          |          |          |       |         |      |
| Directly     | netcoreapp31 | .NET Core 3.1 | 659.4 ns |  6.64 ns |  5.88 ns |  1.00 |    0.00 |    1 |
| QuickStyle   | netcoreapp31 | .NET Core 3.1 | 685.1 ns |  8.25 ns |  7.72 ns |  1.04 |    0.01 |    2 |
| CacheVisitor | netcoreapp31 | .NET Core 3.1 | 655.6 ns |  5.90 ns |  5.52 ns |  0.99 |    0.01 |    1 |
|              |              |               |          |          |          |       |         |      |
| Directly     | netcoreapp5  | .NET Core 5.0 | 624.2 ns |  3.59 ns |  3.00 ns |  1.00 |    0.00 |    2 |
| QuickStyle   | netcoreapp5  | .NET Core 5.0 | 641.2 ns |  5.60 ns |  4.97 ns |  1.03 |    0.01 |    3 |
| CacheVisitor | netcoreapp5  | .NET Core 5.0 | 604.2 ns |  8.19 ns |  7.66 ns |  0.97 |    0.01 |    1 |

结论:

1. 使用 Newbe.ObjectVisitor, 仅仅只用非常少的额外时间消耗就得到了和硬编码完全一样的效果。
2. 使用 Newbe.ObjectVisitor 内置的方法，仅仅需要消耗非常少的额外时间就可以省去自己构建 visitor 的时间。是一种值得参考的编写方式。
