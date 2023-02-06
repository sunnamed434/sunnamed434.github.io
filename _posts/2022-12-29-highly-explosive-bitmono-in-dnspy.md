---
layout: post
title: Highly explosive BitMono in dnSpy
date: '2022-12-29 17:06:59 +0200'
categories: bitmono mono-magic dnspy bugs asmresolver anti-decompiler
---

### Edit 2/5/2023
* This bug was fixed in dnSpyEx 6.3.0 after 1 week after the post, see [commit](https://github.com/dnSpyEx/dnSpy/commit/d58b75265fbd0a767668b5e485219d0faba01dcd).
* Source code with bug fix of dnSpyEx 6.3.0, see [here](https://github.com/dnSpyEx/dnSpy/blob/d58b75265fbd0a767668b5e485219d0faba01dcd/Extensions/dnSpy.Analyzer/TreeNodes/ScopedWhereUsedAnalyzer.cs#L187), fixes were marked with comments.

### What's the plan
* Crash dnSpy while analyzing types same as [BitMono](https://github.com/sunnamed434/BitMono) do

### Compability
* This bug was tested only on [Mono Windows x64 5.10.1](https://download.mono-project.com/archive/5.10.1/windows-installer/)
* Works before dnSpyEx 6.3.0

### Summary
You could say, there are plenty of ways how to crash the dnSpy, I agree with you, but I'll show you one more way, I don’t know, fortunately, or unfortunately, most likely fortunately or maybe not, and maybe yes.

I asked ElektroKill to leave a star on [BitMono](https://github.com/sunnamed434/BitMono) to reveal the secret of the Dark Magic to him, I just wanted to agree on a good, but he rejected me, here's what you did, ElektroKill, you brought me on it. Probably after this post this will be fixed in new versions of dnSpyEx, but, anyway this will take plenty of time to do this, because dnSpy is a very huge app, to fix something you need to fix everything :D, so, use it while this is alive, btw this is always useful in old versions of dnSpy. A bit later after the this post he agreed to leave a star to reveal the secret.

## How it works?
* When we add the nested type to dont set any of the nested type accessibility attributes according to ECMA CIL standard nested types should always have one of them applied.
* According to dnSpy sources, this bug worked well because of the circumstances not suggested, see [code](https://github.com/dnSpyEx/dnSpy/blob/c011dc0c13a1a5c893a30dec1a3766176639a178/Extensions/dnSpy.Analyzer/TreeNodes/ScopedWhereUsedAnalyzer.cs#L187).

### Crash the dnSpy
First of all, create new NestedType and important to DO NOT specify the Public attributes, etc, somehow someone decided to decompile your precious app (this could be your ad: use [BitMono](https://github.com/sunnamed434/BitMono) to prevent such things), so continuing, and habitually trying to analyze where your things was used, there we will catch the hare.

As an example I'll use AsmResolver and implementation from actual [BitMono](https://github.com/sunnamed434/BitMono), which will crash the dnSpy, you could do the same with dnlib, nothing really changes between the code, but with AsmResolver try to write the Module with `Advanced PE Image Building` to bypass possible errors while writing the module (I'm sure, everything is ok, you can write by default).
## Create your own NestedType in Module
* This code adds new nested in Module
```csharp
var moduleType = module.GetOrCreateModuleType();
moduleType.NestedTypes.Add(new TypeDefinition(string.Empty, "CrashdnSpy", TypeAttributes.Sealed | TypeAttributes.ExplicitLayout));
```

## Prepare the all Module nested types
* This code changes the Attributes of all nested types inside a Module
```csharp
var moduleType = module.GetOrCreateModuleType();
foreach (var type in module.GetAllTypes())
{
    if (type.DeclaringType == moduleType && type.IsNested)
    {
        type.Attributes = TypeAttributes.Sealed | TypeAttributes.ExplicitLayout;
    }
}
```

### Let's catch the first hare
Write the module and drag-and-drop/specify the output file into dnSpy, search for the NestedType in Module type, then try to analyze it, and boom! Magic!

![Select type and analyze](/assets/images/highly-explosive-bitmono-in-dnspy/analyze.dark.png){: .dark width="700" height="400" }
![Select analyzation drop-down method](/assets/images/highly-explosive-bitmono-in-dnspy/crash.dark.png){: .dark width="700" height="400" }

![Select type and analyze](/assets/images/highly-explosive-bitmono-in-dnspy/analyze.light.png){: .light width="700" height="400" }
![Select analyzation drop-down method](/assets/images/highly-explosive-bitmono-in-dnspy/crash.light.png){: .light width="700" height="400" }

Congrats! You've catched the hare :)