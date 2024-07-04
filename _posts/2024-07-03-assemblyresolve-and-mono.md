---
layout: post
title: AssemblyResolve And Mono
date: 2024-07-03 15:08 +0300
categories: mono unity assemblyresolve
---

## Why AssemblyResolve in Mono is so fun
Idk why its so fun, but I spent so many time figuring it out. Not many really know about how it work but who knows benefits from that a lot, let's start ;)

## My attempts

Attempts which not helped me:

- Minimize/Remove the difference between versions in NuGet packages (that helped a bit)
- Use auto-binding redirects such as in EF Core/MySQL projects
- Use hands-made binding redirects
- Load the identical assemblies (like filename1.dll; filename2.dll) but with required versions (this caused me exceptions that it could not find a proper method, like missing method exception, etc)
- Install all mono GAC runtime libraries
- Reading/watching tons of content and asking dozens of questions

## Example

Ok, let's say you have Web API and a client (maybe HttpClient/WebClient) which downloads your `List<Data>` it can be a plugins some tools whatever, but here's a case its very important to load it correctly, `Assembly.Load` and `AssemblyResolve` is not enough at all.

![DLL Hell Meme](/assets/images/assemblyresolve-and-mono/dll_hell.png)

## Could not load file or assembly … or one of its dependencies
You loaded all libraries, seems to be all fine, but still it not work.

A little logs:
```console
Could not find dependency..
Could not load signature of..
Could not find type..
```

It's all usual logs. However, sometimes you can see such logs but actually your Assembly can be loaded correctly, but maybe some of your references was resolved but a little later than other `AssemblyResolve` event listeners - which can also cause huge problems.

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

## DO NOT load multiple assemblies
Some of the Unity and Mono versions don't like when you load multiple assemblies which has identic name/version.

As a result you might see errors like:
- Could not load signature of
- Could not find dependency
- "Something about methods"

## DO NOT Reference ANYTHING at ALL
Inside of your plugin (loader for example) do not reference any library, do not use any NuGet Packages, keep it super simple, it will save against most of the problems.

## DO NOT use HttpClient and WebClient
Instead use UnityWebRequest (if you're on Unity), if its basic Mono use WebClient.

## DO NOT target framework as netstandard 
Inside of your plugin (loader for example) which loads the assemblies do not use netstandard as target framework, because it will fire a AssemblyResolve for netstandard library, which we don't need, instead use the same target framework as your game or whatever, for example `.NET Framework`.

If your game or whatever is already using netstandard then all fine.

## v1.1.1 Not Means v1.1.1 of NuGet Package Version
`Could not find dependency: System.Web.ApplicationServices, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35
` you see this message in logs and think Oh yeah! I'll find this library, you go to the NuGet Packages and search the same version, nope, its wrong, remember:
- Version of assembly doesn't means the version of NuGet Package, inside of the NuGet package there could be a different version of assembly.

## Load as earlier as possible
Try to load your libraries, specificially runtime libraries from GAC before any stuff loads, this actually can solve most of the problems.

For example in game like `Unturned` you can make a `Module` which loads before any plugins and it first loads all libraries of this `Module` and then loading the actual `Module`.

## Put Libraries In `\Managed`
It will work only with Unity games.

By default if Unity game can't find Library `On AssemblyResolve` it will try to find and load it from `Managed` directory.

So, you can simply put your Library there.

## `Solutions` not helped
This means something else loading incorrect libraries, or maybe have duplicated libraries, maybe somewhere you can find library as a File stored on a disk and loaded automatically by Mono or maybe by something else, which actually lead to this problem.

## Conclusion
I saw many many many!!! super many people trying to figure it out and having similar problem, that's why I made this post. Even me asked about this many times but didn't get a help, maybe I understand why but anyway.

Using the upper `Solutions` I made it possible to load `EF Core`, `Microsoft.Extensions.DependencyInjection` and 100+ more libraries in Mono, and including a moment that along side with me there are about 100+ other libraries and plugins.