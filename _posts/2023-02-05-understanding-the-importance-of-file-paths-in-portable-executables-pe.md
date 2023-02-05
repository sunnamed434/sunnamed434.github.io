---
layout: post
title: Understanding the Importance of File Paths in Portable Executables (PE)
date: '2023-02-05 13:22:51 +0200'
categories: pe hex dnspy security
---

### The Idea
PE (Portable Executable) stores file paths as part of its metadata to ensure that the file can be loaded and executed properly. The file path information is used by the operating system to locate the file on the file system, and it also helps in resolving any dependencies that the file might have on other files or resources. Additionally, it allows the executable file to be portable, as it can be moved to different locations on the file system without breaking the links to its dependencies.

### Reproduce
1. Install the file, I'll use [AsmResolver 5.0.0](https://github.com/Washi1337/AsmResolver/releases/tag/v5.0.0), download [AsmResolver_v5.0.0_Binaries.zip](https://github.com/Washi1337/AsmResolver/releases/download/v5.0.0/AsmResolver_v5.0.0_Binaries.zip)
2. Extract ZIP, and then open-up `extracted_asmresolver_zip_path\netstandard2.0`
3. Open-up the `AsmResolver.dll` in dnSpy or Hex Editor (Highly recommend you `HxD`)
4. If you're using the dnSpy, press on file, and then press hotkey `CLTR + X`, or right click on mouse `Open Hex Editor`.
5. Search in Hex something seems to the File Path.

![dnSpy Hex file path](/assets/images/understanding-the-importance-of-file-paths-in-portable-executables-pe/hex-path-to-pdb-dark.png){: .dark }
![dnSpy Hex file path](/assets/images/understanding-the-importance-of-file-paths-in-portable-executables-pe/hex-path-to-pdb-light.png){: .light }

```
C:\projects\asmresolver\src\AsmResolver\obj\Release\netstandard2.0\AsmResolver.pdb
```

## How to Protect Yourself from that?
You can do the same through the IDE options, in this case simply I'll change the `.csproj`. 
Open your `.csproj` of your IDE project, and edit it. 
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <!-- Not important to add, just as example -->
  <PropertyGroup>
    <TargetFrameworks>netstandard2.0;net462</TargetFrameworks>
    <LangVersion>10.0</LangVersion>
  </PropertyGroup>

  <!-- Add this for Release, to remove the debug symbols -->
  <PropertyGroup Condition=" '$(Configuration)' == 'Release' ">
    <DebugSymbols>false</DebugSymbols>
    <DebugType>none</DebugType>
  </PropertyGroup>

  <!-- Add this for Debug, to remove the debug symbols -->
  <PropertyGroup Condition=" '$(Configuration)' == 'Debug' ">
    <DebugSymbols>false</DebugSymbols>
    <DebugType>none</DebugType>
  </PropertyGroup>
</Project>
```

Done, check the magic now! =)