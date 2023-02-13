---
layout: post
title: C# Code Optimization Tricks
date: '2023-02-13 12:50:24 +0200'
categories: csharp optimization tricks
---

## About Optimization
This is such a thing, quite complicated in places, and before we discuss it, we need to understand a few things.

The first thing is when we're doing optimization we know how things work, for example, details of implementation, and the specific things, also very often it's easy to do a mistake and add an optimization trick that you found here, doesn't check it out, and your optimization will get even worse than before, on the other hand, it's not hard to do right.

You don't really need to be a .NET Senior to write optimized code for your project, knowing the basics is enough, but sometimes would be useful to be Senior to know the more tricks based on huge background experience.

## Setup
![BenchmarkDotNet Logo](/assets/images/c-code-optimization-tricks/benchmarkdotnet-logo.png)

In case when you want to test the perfomance of something the best and popular way in .NET is to use [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet), also this post is based on this library, you will understand later, why. As an example why this library is so popular, even the [dotnet perfomance](https://github.com/dotnet/performance) uses this library for the benchmarking.

Very simple example of the benchmark:
```csharp
BenchmarkRunner.Run<MathBenchmark>();
```

```csharp
public class MathBenchmark
{
    private int _a;
    private int _b;

    [GlobalSetup]
    public void Setup()
    {
        _a = 10;
        _b = 10;
    }
    
    [Benchmark]
    public int Multiply()
    {
        return _a * _b;
    }
    [Benchmark]
    public int Divide()
    {
        return _a / _b;
    }
}
```

After running it, I got such output:
```
BenchmarkDotNet=v0.13.4, OS=Windows 11 (10.0.22621.1105)
11th Gen Intel Core i5-1135G7 2.40GHz, 1 CPU, 8 logical and 4 physical cores
.NET SDK=7.0.100
  [Host]     : .NET 6.0.11 (6.0.1122.52304), X64 RyuJIT AVX2
  DefaultJob : .NET 6.0.11 (6.0.1122.52304), X64 RyuJIT AVX2
```

```
|   Method |      Mean |     Error |    StdDev | Median |
|--------- |----------:|----------:|----------:|-------:|
| Multiply | 0.1137 ns | 0.0581 ns | 0.1714 ns | 0.0 ns |
|   Divide | 0.1136 ns | 0.0618 ns | 0.1793 ns | 0.0 ns |
```

## Tricks
I want to notice that these tricks are tested on .NET 6.0 - which is actually yet not bad optimized.

I'll use such terminology for rating the tricks from easy-hard, than higher number than harder the tricks and requires knowledge of the how works compiler in depth or other complex things.

So, you after reading the post you could, `Uhm.. like, you have the Hard trick, but this trick is so easy.`, for example the easy - means you doesn't need to know compiler peculiarities, etc. Than higher we are moving with the easy-hard, than more you need to know the context to understand the purpose of the trick, its when you used be to do something many times and you know how to do that better, kinda like best practice.

* Easy (1 +)
* Medium (2 ++)
* Hard (3 +++)

Let's start, you may know these tricks, but, just remind them.

## Easy

### foreach IEnumerable vs List
```csharp
list = Enumerable.Range(0, 100).ToList();
enumerable = list; // IEnumerable<int>
```

```csharp
var result = 0;
foreach (var value in enumerable)
{
    result += value;
}
```

> 610.91 ns
{: .prompt-danger }

```csharp
var result = 0;
foreach (var value in list)
{
    result += value;
}
```

> 84.47 ns
{: .prompt-tip }

### List.Capacity
```csharp
var list = new List<int>();
for (var i = 0; i < 100; i++)
{
    list.Add(i);
}
```

> 365.7 ns
{: .prompt-danger }

```csharp
var list = new List<int>();
list.Capacity = 100;
for (var i = 0; i < 100; i++)
{
    list.Add(i);
}
```

> 221.7 ns
{: .prompt-tip }

### Contains & IEquatable
```csharp
public struct A
{
    public int Value;
}
public struct B : IEquatable<B>
{
    public bool Equals(B other)
    {
        return other.Equals(this);
    }
}

refsA = new List<A>();
refsB = new List<B>();
```

```csharp
refsB.Contains(new B());
```

> 0.6466 ns
{: .prompt-danger }

```csharp
refsA.Contains(new A());
```

> 0.4640 ns
{: .prompt-tip }

## Medium

### NoInlining vs AggressiveInlining
```csharp
[MethodImpl(MethodImplOptions.NoInlining)]
public void NoInlining()
{
}
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public void AggressiveInlining()
{
}
```

```csharp
NoInlining();
```

> 0.4838 ns
{: .prompt-danger }


```csharp
AggressiveInlining();
```

> 0.0332 ns
{: .prompt-tip }

### List foreach vs for
```csharp
values = Enumerable.Range(0, 100).ToList();
```

```csharp
foreach (var value in values)
{
}
```

> 85.20 ns
{: .prompt-danger }

```csharp
for (int i = 0; i < values.Count; i++)
{
}
```

> 32.79 ns
{: .prompt-tip }

### Finalizers
```csharp
public class A
{
}
public class B
{
   ~B()
   {
   }
}
```

```csharp
new B();
```

> 130.9663 ns
{: .prompt-danger }

```csharp
new A();
```

> 0.0349 ns
{: .prompt-tip }


### ArrayPool
```csharp
text = "Hello";
lenght = text.Length;
```

```csharp
var buffer = new byte[256];
var bytesCount = Encoding.UTF8.GetBytes(text, 0, lenght, buffer, 0);
using var memoryStream = new MemoryStream();
memoryStream.Write(buffer, 0, lenght);
var bytes = memoryStream.ToArray();
```

> 92.81 ns (allocated 656 B)
{: .prompt-danger }

```csharp
var buffer = ArrayPool<byte>.Shared
    .Rent(256);
var bytesCount = Encoding.UTF8.GetBytes(text, 0, lenght, buffer, 0);
using var memoryStream = new MemoryStream();
memoryStream.Write(buffer, 0, lenght);
ArrayPool<byte>.Shared.Return(buffer);
var bytes = memoryStream.ToArray();
```

> 108.10 ns (allocated 376 B)
{: .prompt-tip }

### stackalloc
```csharp
var buffer = new byte[256];
var bytesCount = Encoding.UTF8.GetBytes(text, 0, lenght, buf
using var memoryStream = new MemoryStream();
memoryStream.Write(buffer, 0, lenght);
var bytes = memoryStream.ToArray();
```

> 96.92 ns (allocated 656 B)
{: .prompt-danger }

```csharp
Span<byte> buffer = stackalloc byte[256];
var bytesCount = Encoding.UTF8.GetBytes(text, buffer);
using var memoryStream = new MemoryStream();
memoryStream.Write(buffer.Slice(0, bytesCount));
var bytes = memoryStream.ToArray();
```

> 78.81 ns (allocated 376 B)
{: .prompt-tip }

## Hard

### Delegate.CreateDelegate
```csharp
public class API
{
    public static void StaticVoidMethod()
    {
    }
}

methodInfo = typeof(API).GetMethod(nameof(API.StaticVoidMethod));
@delegate = (Action)Delegate.CreateDelegate(typeof(Action), methodInfo);

// If you need a method with a return type use (Func<T>)Delegate.CreateDelegate((Func<T>), methodInfo)
```

```csharp
methodInfo.Invoke(null, Array.Empty<object>());
```

> 39.890 ns
{: .prompt-danger }

```csharp
@delegate.Invoke();
```

> 1.593 ns
{: .prompt-tip }

### Unsafe.As
```csharp
public class A
{
}

@object = new A();
```

```csharp
var obj = (A)@object;
```

> 0.5847 ns
{: .prompt-danger }

```csharp
var obj = Unsafe.As<object, A>(ref @object);
```

> 0.2757 ns
{: .prompt-tip }