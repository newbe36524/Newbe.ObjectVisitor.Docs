---
title: Cache vs None Cached Object Visitor
description:
published: true
date: 2020-11-26T14:51:47.587Z
tags:
editor: undefined
dateCreated: 2020-11-26T14:46:40.417Z
---

### Cache vs None Cached Object Visitor

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

| Method          | Description              |
| --------------- | ------------------------ |
| CacheVisitor    | Cache Visitor            |
| NoCacheVisitor  | Do not cache the Visitor |
| ReflectProperty | To use reflection        |

Chart:

![newbe.objectvisitor.benchmarktest.cachevisitortest-barplot.png](/benchmark/newbe.objectvisitor.benchmarktest.cachevisitortest-barplot.png)

Data:

| Method          | Job          | Runtime       |         Mean |       Error |      StdDev |    Ratio | RatioSD | Rank |
| --------------- | ------------ | ------------- | ------------:| -----------:| -----------:| --------:| -------:| ----:|
| CacheVisitor    | net461       | .NET 4.6.1    |     783.2 ns |     5.34 ns |     5.00 ns |     1.00 |    0.00 |    1 |
| ReflectProperty | net461       | .NET 4.6.1    |   1,528.1 ns |    16.26 ns |    14.41 ns |     1.95 |    0.02 |    2 |
| NoCacheVisitor  | net461       | .NET 4.6.1    | 988,465.0 ns | 6,373.48 ns | 5,649.92 ns | 1,261.57 |   11.70 |    3 |
|                 |              |               |              |             |             |          |         |      |
| CacheVisitor    | net48        | .NET 4.8      |     809.0 ns |    13.99 ns |    13.08 ns |     1.00 |    0.00 |    1 |
| ReflectProperty | net48        | .NET 4.8      |   1,531.4 ns |    23.96 ns |    22.41 ns |     1.89 |    0.03 |    2 |
| NoCacheVisitor  | net48        | .NET 4.8      | 996,612.1 ns | 9,277.44 ns | 8,678.12 ns | 1,232.28 |   24.07 |    3 |
|                 |              |               |              |             |             |          |         |      |
| CacheVisitor    | netcoreapp21 | .NET Core 2.1 |     793.3 ns |     6.11 ns |     5.71 ns |     1.00 |    0.00 |    1 |
| ReflectProperty | netcoreapp21 | .NET Core 2.1 |   1,425.3 ns |    26.15 ns |    25.69 ns |     1.80 |    0.04 |    2 |
| NoCacheVisitor  | netcoreapp21 | .NET Core 2.1 | 724,556.5 ns | 6,290.47 ns | 5,576.34 ns |   913.33 |   11.43 |    3 |
|                 |              |               |              |             |             |          |         |      |
| CacheVisitor    | netcoreapp31 | .NET Core 3.1 |     636.8 ns |     2.30 ns |     2.15 ns |     1.00 |    0.00 |    1 |
| ReflectProperty | netcoreapp31 | .NET Core 3.1 |   1,294.0 ns |     6.91 ns |     5.77 ns |     2.03 |    0.01 |    2 |
| NoCacheVisitor  | netcoreapp31 | .NET Core 3.1 | 625,524.4 ns | 1,596.90 ns | 1,333.49 ns |   982.17 |    3.79 |    3 |
|                 |              |               |              |             |             |          |         |      |
| CacheVisitor    | netcoreapp5  | .NET Core 5.0 |     629.6 ns |     3.81 ns |     3.56 ns |     1.00 |    0.00 |    1 |
| ReflectProperty | netcoreapp5  | .NET Core 5.0 |   1,211.6 ns |    14.09 ns |    13.18 ns |     1.92 |    0.02 |    2 |
| NoCacheVisitor  | netcoreapp5  | .NET Core 5.0 | 545,642.6 ns | 1,633.25 ns | 1,447.83 ns |   866.61 |    4.45 |    3 |

Conclusions:

1. It takes some time to build an ObjectVisitor, because it needs to build some objects and it needs to be reflected.So we recommend caching the ObjectVisitor cache for use.Of course, in some scenarios where performance is not sensitive, it doesn't matter if the cache is not cached, after all, this construction process is less than a millisecond.
2. The cached ObjectVisitor is much faster than the reflection.
