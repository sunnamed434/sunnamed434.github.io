---
layout: post
title: Highly explosive BitMono in dnSpy
date: '2022-12-29 17:06:59 +0200'
categories: bitmono mono-magic dnspy bugs asmresolver
---

~~UPDATE 12/30/2022:
How are you down there? ElektroKill made a super-mega-fast hot-fix of this bug and some other yesterday right after hours of post, in dnSpy v6.3.0-rc1, seem like he said in the release notes that calli were fixed, sadly, but not yet :D. So, time is gone, however old versions are still fine! ;)
So, this is outdated at some point, depending on what dnSpy version is being used.~~

UPDATE 12/30/2022:
After a 30 minutes I understand that I was wrong with reproducing the bug, so this is still working!

First of all, leave a star on [BitMono](https://github.com/sunnamed434/BitMono), because [BitMono](https://github.com/sunnamed434/BitMono) says a happy new year and does open-source magic and describes how to even do the same and don't open the source code, lol, or I'll come for you, isn't that real magic?
Today, I'll show you how to [BitMono](https://github.com/sunnamed434/BitMono) crashes dnSpy/dnSpyEx 6.20 - all of these precious things, also there's one more known bug and related to RVA.

* These bugs are compatible only with Mono but good to know them as additional complex knowledge

You could say, there are plenty of ways how to crash it, I agree with you, but I'll show you one more way, I don’t know, fortunately, or unfortunately, most likely fortunately or maybe not, and maybe yes.

I asked ElektroKill to leave a star on [BitMono](https://github.com/sunnamed434/BitMono) to reveal the secret of the Dark Magic to him, I just wanted to agree on a good, but he rejected me, here's what you did, ElektroKill, you brought me on it. Probably after this post this will be fixed in new versions of dnSpyEx, but, anyway this will take plenty of time to do this, because dnSpy is a very huge app, to fix something you need to fix everything :D, so, use it while this is alive, btw this is always useful in old versions of dnSpy.

Today's goal is to crash dnSpy while analyzing types, for example, somehow someone decided to decompile your precious app (this could be your ad: use [BitMono](https://github.com/sunnamed434/BitMono) to prevent such things), so continuing, and habitually trying to analyze where your magic was used, here we will catch the hare.

As an example I'll use AsmResolver and custom injection of byte[] array from [BitMono](https://github.com/sunnamed434/BitMono) in the app, which will crash the dnSpy, you could do the same with dnlib, nothing really changes between the code, but with AsmResolver try to write the Module with `Advanced PE Image Building` to bypass possible errors while writing the module.
Let's start with what, you need to use this method:
```csharp
public FieldDefinition InjectInvisibleArray(ModuleDefinition module, TypeDefinition type, byte[] data, string name)
{
    var valueType = module.DefaultImporter.ImportType(typeof(ValueType));
    var classWithLayout = new TypeDefinition(null, "<>c", TypeAttributes.Sealed | TypeAttributes.ExplicitLayout, module.DefaultImporter.ImportType(valueType))
    {
        ClassLayout = new ClassLayout(0, (uint)data.Length),
    };
    var compilerGeneratedAttribute = InjectCompilerGeneratedAttribute(module);
    classWithLayout.CustomAttributes.Add(compilerGeneratedAttribute);
    type.NestedTypes.Add(classWithLayout);

    var fieldWithRVA = new FieldDefinition("<>c", FieldAttributes.Assembly | FieldAttributes.Static | FieldAttributes.HasFieldRva | FieldAttributes.InitOnly, new FieldSignature(classWithLayout.ToTypeSignature()));
    fieldWithRVA.FieldRva = new DataSegment(data);
    classWithLayout.Fields.Add(fieldWithRVA);

    var byteArray = module.DefaultImporter.ImportType(typeof(byte[]));
    var fieldInjectedArray = new FieldDefinition(name, FieldAttributes.Assembly | FieldAttributes.Static, new FieldSignature(byteArray.ToTypeSignature()));
    classWithLayout.Fields.Add(fieldInjectedArray);

    var systemByte = module.DefaultImporter.ImportType(module.CorLibTypeFactory.Byte.ToTypeDefOrRef());
    var initializeArrayMethod = module.DefaultImporter.ImportMethod(typeof(RuntimeHelpers).GetMethod(nameof(RuntimeHelpers.InitializeArray), new Type[]
    {
        typeof(Array),
        typeof(RuntimeFieldHandle)
    }));

    var cctor = classWithLayout.GetOrCreateStaticConstructor();
    var instructions = cctor.CilMethodBody.Instructions;
    instructions.InsertRange(0, new CilInstruction[]
    {
        new CilInstruction(CilOpCodes.Ldc_I4, data.Length),
        new CilInstruction(CilOpCodes.Newarr, systemByte),
        new CilInstruction(CilOpCodes.Dup),
        new CilInstruction(CilOpCodes.Ldtoken, module.DefaultImporter.ImportField(fieldWithRVA)),
        new CilInstruction(CilOpCodes.Call, initializeArrayMethod),
        new CilInstruction(CilOpCodes.Stsfld, fieldInjectedArray),
    });
    return fieldInjectedArray;
}
```

After injection of byte[] array with `InjectInvisibleArray`, write the module and drag-and-drop/specify the output file into dnSpy, search for injected type, then try to analyze it, boooom! Maaagic!

![Select type and analyze](/assets/images/highly-explosive-bitmono-in-dnspy/analyze.dark.png){: .dark }
![Select analyzation drop-down method](/assets/images/highly-explosive-bitmono-in-dnspy/crash.dark.png){: .dark }

You could don't see these types because they're also hiden as compiler generated, you've to to enabled some things in your dnSpy.
Open-up options in dnSpy
![open-options](/assets/images/highly-explosive-bitmono-in-dnspy/open-options.dark.png){: .dark }
Select decompiler page and stay checked show hidden compiler generated types and methods
![show-hiden-things](/assets/images/highly-explosive-bitmono-in-dnspy/show-hiden-things.dark.png){: .dark }