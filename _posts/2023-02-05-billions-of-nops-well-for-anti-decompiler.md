---
layout: post
title: Billions of Nops - Well for Anti Decompiler?
date: '2023-02-05 12:17:12 +0200'
categories: dnspy bugs asmresolver anti-decompiler
---

### Let's analyze
Say, you’ve opened another crackme or something else that has anti-decompiler protection with tons even if not billions instructions, for example, `nop`, you will say, `Ah, man, I'll just write the static deobfuscator, lol, are you kidding me?` - answering the question, yeah, you’re probably right, but, what if there’s used different instructions, not just `nop`, what to do then? What if you were digging potatoes in the garden today and you're very tired and lazy, and you don’t want to run IDE or create a `.cs` file and then compile it or even write any code now, in this case, I have a better plan for you.

## Recreate the Anti-Decompiler
With such a small code, anyone can prevent you from decompiling, this is sad, isn’t it, yeah?

* This code just adds a new `public static void AntiDnspy() { }` in the Module and then adds 100.000 `nop` instructions, and `ret` at the end.
```csharp
var moduleType = context.Module.GetOrCreateModuleType();
var factory = context.Module.CorLibTypeFactory;
var method = new MethodDefinition("AntiDnSpy", MethodAttributes.Public | MethodAttributes.Static,
    MethodSignature.CreateStatic(factory.Void));
moduleType.Methods.Add(method);
var body = method.CilMethodBody = new CilMethodBody(method);
body.Instructions.Add(new CilInstruction(CilOpCodes.Ret));
for (var i = 0; i < 100000; i++)
{
    body.Instructions.Insert(0, new CilInstruction(CilOpCodes.Nop));
}
```

## Make sure the Anti-Decompiler works fine
Yep, this is crashed... well done!

![dnSpy Crash](/assets/images/billions-of-nops-well-for-anti-decompiler/nops-antidecompiler-crash-dark.png){: .dark }
![dnSpy Crash](/assets/images/billions-of-nops-well-for-anti-decompiler/nops-antidecompiler-crash-light.png){: .light }

## The Better Plan
1. Install old version of [dnSpy 6.1.8 win-32](https://github.com/dnSpy/dnSpy/releases/tag/v6.1.8), very important to install 32-bit version!
2. Open-up your file there, drag-and-drop, whatever you want.
3. Find the Method.
4. The thing you'll see that there's `System.OutOfMemoryException` - and this is what we're looking for, oh yeah :)
5. Press on the method `Edit Method Body...`.
6. A bit laggy, wait 1-2 seconds.. (this is more better than crash or lag for a minute in newest dnSpyEx versions)
7. Magic! We can see the IL code, so now, you can remove the things you don't like, or even just read the IL code.

## Looks better, right?
![dnSpy Anti Decompiler fix](/assets/images/billions-of-nops-well-for-anti-decompiler/nops-antidecompiler-fix-dark.png){: .dark }
![dnSpy Anti Decompiler fix](/assets/images/billions-of-nops-well-for-anti-decompiler/nops-antidecompiler-fix-light.png){: .light }

![dnSpy Anti Decompiler Method body fix](/assets/images/billions-of-nops-well-for-anti-decompiler/nops-antidecompiler-methodbody-dark.png){: .dark }
![dnSpy Anti Decompiler Method body fix](/assets/images/billions-of-nops-well-for-anti-decompiler/nops-antidecompiler-methodbody-light.png){: .light }