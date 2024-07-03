---
layout: post
title: AssemblyResolve And Mono
date: 2024-07-03 15:08 +0300
categories: mono unity assemblyresolve
---

## Why AssemblyResolve in Mono is so fun
Idk why its so fun, but I spent so many time figuring it out, not many really know about this but who knows benefits from that a lot, let's start ;)

Ok, let's say you have Web API which downloads your `List<Data>` it can be a plugins some tools whatever, but here's a case its very important to load it correctly, `Assembly.Load` and `AssemblyResolve` is not enough at all.

![DLL Hell Meme](/assets/images/assemblyresolve-and-mono/dll_hell.png)

## Could not load file or assembly … or one of its dependencies
You loaded all libraries, seems to be all fine, but still it not work.

A little logs:
```console
Could not find dependency..
Could not load signature of..
Could not find type..
```

It's all usual logs.

## Solutions

### Use Mono/Unity GAC Libraries
Instead of copying libraries from building application or searching them on NuGet or other places, first make sure to use libraries from Mono/Unity GAC.

To find the GAC, you need to install appropriate Mono version or just install Unity which your game have or whatever.

For example where GAC can be stored: `Program Files\Mono\lib\mono\gac`

### Patch AppDomain.DoAssemblyResolve
You can try to patch `AppDomain.DoAssemblyResolve` to load your stuff as first as possible, to resolve conflicts with other libraries and etc, but not always it will work.

### Fix your `AssemblyResolve`
First make sure to store somewhere your file name and byte[] of this file which you plan to load (to use it on AssemblyResolve).

```csharp
public class Plugin
{
    public byte[] Content { get; set; }
    public List<Reference> References { get; set; }
}
public class Reference
{
    public string Name { get; set; } // very important to store this name of the file
    public byte[] Content { get; set; }
}
```

```csharp
var response = // GET request for example using UnityWebRequest or any other. 
var plugin = new Plugin
{
    Content = // For example your main file which runs everything (which is dependent on this libraries), which you for example will run later via reflection or any other way
    References = // get the references (libraries)
};
```

Now, the most important part:

```csharp
AppDomain.CurrentDomain.AssemblyResolve += OnAssemblyResolve;
HashSet<Assembly> LoadedAssemblies = [];

var references = plugin.References;
for (var i = 0; i < references.Count; i++)
{
    var reference = references[i];
    if (LoadedAssemblies.FirstOrDefault(
            x => x.GetName().Name == reference.Name) != null)
    {
        // don't load assembly if its loaded yet, since assembly resolve figured it out yet, or its just a duplicate
        continue;
    }
    var assembly = Assembly.Load(reference.Content);
    LoadedAssemblies.Add(assembly);
}

private static Assembly? OnAssemblyResolve(object sender, ResolveEventArgs args)
{
    return FindAssembly(args);
}
private static Assembly? FindAssembly(ResolveEventArgs args)
{
    var assembly = FindLoadedAssembly(args)
                   ?? TryLoadNotLoadedAssembly(args);
    return assembly;
}
private static Assembly? FindLoadedAssembly(ResolveEventArgs args)
{
    var assemblyName = new AssemblyName(args.Name);
    return LoadedAssemblies.FirstOrDefault(
        x => x.GetName().Name == assemblyName.Name);
}
/// <summary>
/// This way we solve a problem when Library dependent on another Library.
/// And its very important to load it on first Assembly Resolve because
/// somehow if you don't resolve it first time it will break everything.
/// </summary>
private static Assembly? TryLoadNotLoadedAssembly(ResolveEventArgs args)
{
    var assemblyName = new AssemblyName(args.Name);
    var reference = plugin.References.FirstOrDefault(x =>
        assemblyName.Name == x.Name);
    if (reference != null)
    {
        var assembly = Assembly.Load(reference.Content);
        LoadedAssemblies.Add(assembly);
        return assembly;
    }
    return null;
}
```

## DO NOT Reference ANYTHING at ALL
Inside of your plugin (loader for example) do not reference any library, do not use any NuGet Packages, keep it super simple, it will save against most of the problems.

## DO NOT use HttpClient and WebClient
Instead use UnityWebRequest (if you're on Unity), if its basic Mono use WebClient.

## DO NOT target framework as netstandard 
Inside of your plugin (loader for example) which loads the assemblies do not use netstandard as target framework, because it will fire a AssemblyResolve for netstandard library, which we don't need, instead use the same target framework as your game or whatever, for example `.NET Framework`.

If your game or whatever is already using netstandard then all fine.

## `Solutions` not helped
This means something else loading incorrect libraries, or maybe have duplicated libraries, maybe somewhere you can find library as a File stored on a disk and loaded automatically by Mono or maybe by something else, which actually lead to this problem.

## Conclusion
I saw many many many!!! super many people trying to figure it out and having similar problem, that's why I made this post. Even me asked about this many times but didn't get a help, maybe I understand why but anyway.