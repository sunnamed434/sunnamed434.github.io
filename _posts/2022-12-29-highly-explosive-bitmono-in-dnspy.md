---
layout: post
title: Highly explosive BitMono in dnSpy
date: '2022-12-29 17:06:59 +0200'
categories: bitmono mono-magic dnspy bugs asmresolver
---

### Edit
* This bug was fixed in dnSpyEx 6.3.0 after 1 week after the post, see [commit](https://github.com/dnSpyEx/dnSpy/commit/d58b75265fbd0a767668b5e485219d0faba01dcd).
* Source code with bug fix of dnSpyEx 6.3.0, see [here](https://github.com/dnSpyEx/dnSpy/blob/d58b75265fbd0a767668b5e485219d0faba01dcd/Extensions/dnSpy.Analyzer/TreeNodes/ScopedWhereUsedAnalyzer.cs#L187), fixes were marked with comments.

Today, I'll show you how to [BitMono](https://github.com/sunnamed434/BitMono) crashes dnSpy/dnSpyEx 6.20

### Compability
* These bugs are compatible only with Mono.
* Works until dnSpyEx 6.3.0
* You can still use the bug because old versions still used!

### Summary
You could say, there are plenty of ways how to crash it, I agree with you, but I'll show you one more way, I don’t know, fortunately, or unfortunately, most likely fortunately or maybe not, and maybe yes.

I asked ElektroKill to leave a star on [BitMono](https://github.com/sunnamed434/BitMono) to reveal the secret of the Dark Magic to him, I just wanted to agree on a good, but he rejected me, here's what you did, ElektroKill, you brought me on it. Probably after this post this will be fixed in new versions of dnSpyEx, but, anyway this will take plenty of time to do this, because dnSpy is a very huge app, to fix something you need to fix everything :D, so, use it while this is alive, btw this is always useful in old versions of dnSpy.

### Today's goal
Today's goal is to crash dnSpy while analyzing types, for example, create new NestedType and important to DO NOT specify the Public attributes, etc, somehow someone decided to decompile your precious app (this could be your ad: use [BitMono](https://github.com/sunnamed434/BitMono) to prevent such things), so continuing, and habitually trying to analyze where your magic was used, here we will catch the hare.

As an example I'll use AsmResolver and implementation from actual [BitMono](https://github.com/sunnamed434/BitMono), which will crash the dnSpy, you could do the same with dnlib, nothing really changes between the code, but with AsmResolver try to write the Module with `Advanced PE Image Building` to bypass possible errors while writing the module (I'm sure, everything is ok, you can write by default).
### Create your own NestedType in Module
```csharp
var moduleType = module.GetOrCreateModuleType();
moduleType.NestedTypes.Add(new TypeDefinition(string.Empty, "CrashdnSpy", TypeAttributes.Sealed | TypeAttributes.ExplicitLayout));
```

### Prepare the all Module nested types
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

### How it works?
* When we add the nested type to dont set any of the nested type accessibility attributes according to ECMA CIL standard nested types should always have one of them applied.
* According to dnSpy sources, this bug was worked well because of the circumstances not suggested, see [code](https://github.com/dnSpyEx/dnSpy/blob/c011dc0c13a1a5c893a30dec1a3766176639a178/Extensions/dnSpy.Analyzer/TreeNodes/ScopedWhereUsedAnalyzer.cs#L187).

### Let's catch the first hare
Write the module and drag-and-drop/specify the output file into dnSpy, search for the NestedType in Module type, then try to analyze it, boooom! Maaagic!

![Select type and analyze](/assets/images/highly-explosive-bitmono-in-dnspy/analyze.dark.png){: .dark }
![Select analyzation drop-down method](/assets/images/highly-explosive-bitmono-in-dnspy/crash.dark.png){: .dark }

You could don't see these types because they're also hiden as compiler generated, you've to to enabled some things in your dnSpy.
Open-up options in dnSpy
![open-options](/assets/images/highly-explosive-bitmono-in-dnspy/open-options.dark.png){: .dark }
Select decompiler page and stay checked show hidden compiler generated types and methods
![show-hiden-things](/assets/images/highly-explosive-bitmono-in-dnspy/show-hiden-things.dark.png){: .dark }

Congrats! You've catched the hare :)