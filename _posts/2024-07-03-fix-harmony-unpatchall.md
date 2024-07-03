---
layout: post
title: Fix Harmony.UnpatchAll
date: 2024-07-03 14:25 +0300
categories: csharp dotnet harmonylib harmony
---

## The problem
Have you ever noticed that [Harmony.UnpatchAll](https://github.com/pardeike/Harmony/blob/efb41b4b3fe24c8a17b40b16508e5da4b5b05002/Harmony/Public/Harmony.cs#L181) method unpatches everything, even patches which made in other assemblies if you don't pass `harmonyID`?

```csharp
public void UnpatchAll(string harmonyID = null)
```

This could lead to a real headace and problems, let's say you or someone's else have a plugin and at some point its unloaded/disposed and do this:

```csharp
/// Called when plugin is unloading or disposing
void UnloadPlugin()
{
    harmony.UnpatchAll(); // Don't do it, it unpatches everything
}
```

You see? It will just unpatch everything, other plugins and your.

## Solution

### Pass the ID of your Harmony instance
Instead, pass the Id of your harmony instance to unpatch only your own patches, but still it might not work, recommend looking next solution.

```csharp
/// Called when plugin is unloading or disposing
void UnloadPlugin()
{
    harmony.UnpatchAll(harmony.Id);
}
```

### Patch `Harmony.UnpatchAll` and force ID
Upper solution might not work because you might have a lot of plugins even not made by yourself, so in this case we will patch the [Harmony.UnpatchAll](https://github.com/pardeike/Harmony/blob/efb41b4b3fe24c8a17b40b16508e5da4b5b05002/Harmony/Public/Harmony.cs#L181) to fix this problem everywhere.

```csharp
/// <summary>
/// Caution for developers: Using `Harmony.UnpatchAll()` without specifying a Harmony ID will unpatch methods across all assemblies, not just your own, this is a solution for this problem.
/// </summary>
[HarmonyPatch]
internal static class PatchHarmony
{
    [HarmonyPrefix]
    [HarmonyPatch(typeof(Harmony), nameof(Harmony.UnpatchAll))]
    private static void Prefix(Harmony __instance, ref string harmonyID)
    {
        if (harmonyID == null) // or you can use instead `string.IsNullOrEmpty(harmonyId)` or `string.IsNullOrWhiteSpace(harmonyId)`
        {
            harmonyID = __instance.Id;
        }
    }
}
```