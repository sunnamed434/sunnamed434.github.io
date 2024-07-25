---
layout: post
title: Ambiguous reference JetBrains.Annotations.UsedImplicitlyAttribute
date: 2024-07-25 15:59 +0300
categories: jetbrains annotations unity csharp dotnet ide
---

## The problem

Do you get an error like this when working with Unity? 

```
Ambiguous reference: JetBrains.Annotations.UsedImplicitlyAttribute JetBrains.Annotations.UsedImplicitlyAttribute match
```

It's normal practice, specifically it happens when working with Unity, however after some research I didn't get a workaround/solution for that, but well I found one solution which is different of what I'll show so I was only able to use outdated old version of UsedImplicitlyAttribute, which is coming from Unity, not from JetBrains itself, it means I couldn't use `ImplicitUseTargetFlags.WithInheritors` option.

## Solution

#### 1. Reference `JetBrains.Annotations` package in `.csproj`

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="JetBrains.Annotations" Version="2024.2.0" /> <!-- This is need to be added in your .csproj -->
  </ItemGroup>

</Project>
```

#### 2. Create a file in your project/solution named `UnityEngineDependentProjectProps.props` or any other name and paste there this content:

```
<!--
Fixes problem when using JetBrains.Annotations in a project that references UnityEngine.CoreModule.
Example of the problem:
Ambiguous reference: JetBrains.Annotations.UsedImplicitlyAttribute JetBrains.Annotations.UsedImplicitlyAttribute match
-->

<Project>
    <ItemGroup>
        <Reference Include="UnityEngine.CoreModule" />
        <Reference Include="JetBrains.Annotations">
            <Aliases>JetBrainsAnnotations</Aliases>
        </Reference>
    </ItemGroup>
</Project>
```

#### 3. Reference `UnityEngineDependentProjectProps.props` in your `.csproj`.

I store the `UnityEngineDependentProjectProps.props` file in the `root\props` of my solution to just in case able to reference it in other projects of my solution.

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
  </PropertyGroup>

  <Import Project="$(MSBuildThisFileDirectory)..\props\UnityEngineDependentProjectProps.props"/>

</Project>
```

#### 4. Use alias and namespace

In every file make sure to have that, and make sure to keep the `extern alias JetBrainsAnnotations` always first (even before usings), otherwise project won't compile.

```csharp
extern alias JetBrainsAnnotations;
using JetBrainsAnnotations::JetBrains.Annotations;
using System; // optional, just example
using UnityEngine; // no problem, we can use Unity here also

class SomeCode {}
// etc
```

Or you can do that in your `GlobalUsings.cs`

```csharp
extern alias JetBrainsAnnotations;
global using JetBrainsAnnotations::JetBrains.Annotations;
global using System; // optional, just example
global using UnityEngine; // no problem, we can use Unity here also
```

## Conclusion

Finally, we can use `[UsedImplicitly(ImplicitUseTargetFlags.WithInheritors)]` =)

The only main thing that motivated me to figure it out is to able use new version of `ImplicitUseTargetFlags`, instead of outdated Unity's one.