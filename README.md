# Building Instructions

## Variables

Change `<mach2 repo directory>` into the Directory where you cloned mach2. Example: `C:\Users\Test1\source\repos\mach2`

## Visual Studio 2022 Components installed:

Desktop Development with C++

Windows Universal CRT SDK

Windows 10 SDK (10.0.19041.0)

Windows Universal C Runtime

WIndows Driver Kit (10.0.22621.382)

## VCPKG Commands:

vcpkg Update Baseline: `vcpkg.exe x-update-baseline --add-initial-baseline`

vcpkg install: `vcpkg.exe install --feature-flags=manifests,binarycaching --triplet "x64-windows"`

## Fix Microsoft.Taef Dependency being missing

#### Tools -> Options -> NuGet Package Manager -> Package Source -> New Package Source (Plus Button):

`https://pkgs.dev.azure.com/ms/ProjectReunion/_packaging/ProjectReunion-Dependencies/nuget/v3/index.json`

(Required because the Microsoft.Taef Dependency with Version 10.58.210222006-develop is otherwise missing)

#### Tools -> NuGet Package Manager -> Package Manager Console:

Press on the "Restore" Button, otherwise use `Install-Package Microsoft.Taef -version 10.58.210222006-develop`

## Include capstone.dll in the Output Directory

#### mach2-cli -> Build Events -> Post-Build Event -> Command Line:

```
copy /y "$(ProjectDir)features.txt" "$(OutDir)"
copy /y "$(CurrentVsInstallRoot)\DIA SDK\bin\amd64\msdia140.dll" "$(OutDir)"
copy /y "$(ProjectDir)vcpkg_installed\x64-windows\bin\capstone.dll" "$(OutDir)"
```

## Building

#### Solution -> Retarget Solution: Windows SDK Version: `10.0.19041.0`

#### mach2-cli -> Properties -> C/C++ -> General -> Additional Include Directories: 

`C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\ucrt;<mach2 repo directory>\vcpkg_installed\x64-windows\include`

#### mach2-cli -> Properties -> Linker -> Input -> Additional Dependencies: 

`mach2-core.lib;<mach2 repo directory>\vcpkg_installed\x64-windows\lib\capstone.lib;rpcrt4.lib;dbghelp.lib;comsuppw.lib;pathcch.lib;diaguids.lib;ntdll.lib;kernel32.lib;user32.lib;gdi32.lib;winspool.lib;comdlg32.lib;advapi32.lib;shell32.lib;ole32.lib;oleaut32.lib;uuid.lib;odbc32.lib;odbccp32.lib;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\ucrt\x64\libucrtd.lib;%(AdditionalDependencies)`

#### mach2-core -> Properties -> C/C++ -> General -> Additional Include Directories: 

`C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\ucrt;<mach2 repo directory>\vcpkg_installed\x64-windows\include`

#### mach2-test -> Properties -> C/C++ -> General -> Additional Include Directories: 

`C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\ucrt;<mach2 repo directory>\vcpkg_installed\x64-windows\include`

#### mach2-test -> Properties -> Linker -> Input -> Additional Dependencies: 

`mach2-core.lib;<mach2 repo directory>\vcpkg_installed\x64-windows\lib\capstone.lib;rpcrt4.lib;dbghelp.lib;comsuppw.lib;pathcch.lib;diaguids.lib;ntdll.lib;kernel32.lib;user32.lib;gdi32.lib;winspool.lib;comdlg32.lib;advapi32.lib;shell32.lib;ole32.lib;oleaut32.lib;uuid.lib;odbc32.lib;odbccp32.lib;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\ucrt\x64\libucrtd.lib;%(AdditionalDependencies)`

## Changes made to Source Code

#### mach2-cli -> mach2.rc: Version: VS_VERSION_INFO:

| Key | Value |
| --- | --- |
| FILEVERSION | 0, 8, 0, 0 |
| PRODUCTVERSION | 0, 8, 0, 0 |
| FileVersion | 0.8.0.0 |
| ProductVersion | Commit e20cf4d |

---

# Mach2

![](./gfx/usage.png)

*Mach2* manages the Windows Feature Store, where Features (and associated on/off state) live. This store lives in the undocumented Windows Notification Facility (WNF), which provides publish-subscribe messaging for kernel components, system services, and user-space applications.

Windows currently contains **thousands** of Feature switches that turn on and off new and unfinished functionality, mitigations, test hooks, and overrides. *Mach2* provides facilities to discover these switches and turn them on or off.

Without going into specifics, *Mach2* commands generally fall into one of two buckets:

### Scanning
*Mach2* operates on Feature IDs for the bulk of its operations. But finding interesting Features to turn on and off can be a chore, so it includes a scanning function. This function scans Microsoft Program Database (PDB) files for Feature symbols and collects them for review. A user can then review the results and cherry pick which Features warrant further investigation.

### Management
*Mach2* can dump the current Feature Control store and resolve known IDs to names for convienence. (It reads simple key:value pairs from `features.txt` on disk.)

With a Feature ID in hand, *Mach2* can *enable* or *disable* a Feature on the local system. Both of these actions create configuration state for the Feature and set the feature to *Enabled* or *Disabled* respectively. The user can also choose to *Revert* back to the default configuration -- that is, let the Feature turn itself on or off as desired. (There is a *Default* configuration state that could be set, the tool currently opts to remove reverted features from the configuration store altogether.)

While the tool can manipulate Feature states, the Feature itself drives state compliance. That is, it can choose to ignore its configured state. Various factors, including what's referred to internally as a *staging* configuration, can dictate whether a Feature respects its configurable state or not. (*Always Disabled* staged Features, for example, are crippled/stripped during Windows build compilation and cannot be turned on with Feature Control.)

## Install
Installation is not required, however *Mach2* utilizes registration-free COM activation to bring in DIA SDK components so `msdia120.dll` must be present.

## Building from source

Compilation requires Visual Studio 2017 or newer, a recent version of the Windows 10 SDK installed, [and vcpkg](https://docs.microsoft.com/en-us/cpp/build/install-vcpkg).

### Step-by-step
1. `vcpkg --feature-flags=manifests install` in the mach2 root folder
2. Open `mach2.sln` in Visual Studio
3. Change the target platform and configuration as needed
4. Build

## Usage
*Mach2* relies on [CLI11](https://github.com/CLIUtils/CLI11) to provide a canonical command line argument-driven interface. It's recommended you run the tool with `--help` for details on how to use the tool.

## Testimonials

### Microsoft

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Please never use this tool. The ways in which it can hose your system (and also defeat the purpose of WIP) are myriad.</p>&mdash; Brandon Paddock (@BrandonLive) <a href="https://twitter.com/BrandonLive/status/1012145159104954368?ref_src=twsrc%5Etfw">June 28, 2018</a> <a href="https://web.archive.org/web/20180628053245/https://twitter.com/BrandonLive/status/1012145159104954368">(archived)</a></blockquote>

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">You can cite me saying this. Hacking around in internal system state you don’t understand is very risky, and creates an inconsistent state that shouldn’t normally be possible. Plus mucks up data and wastes people’s time.</p>&mdash; Brandon Paddock (@BrandonLive) <a href="https://twitter.com/BrandonLive/status/1012163763204583425?ref_src=twsrc%5Etfw">June 28, 2018</a> <a href="https://web.archive.org/web/20180628051527/https://twitter.com/BrandonLive/status/1012163763204583425">(archived)</a></blockquote>

## Contributions
Contributions are greatly appreciated. Keep the license in mind (GPLv3).

Try to ping me in advance if you're working on any major changes to ensure we don't clash.
