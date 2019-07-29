# GhidraDec: Ghidra Decompiler Plugin for IDA Pro

GhidraDec: Ghidra decompiler plugin for Hex-Rays IDA (Interactive DisAssembler) Pro

The plugin is compatible with the IDA 5/6/7.x versions.
The plugin does NOT work with the freeware version of IDA 7.0.
The plugin comes at both 32-bit and 64-bit address space variants (both are 64-bit binaries). I.e. it works in both `ida` and `ida64`.
It can decompile any processor architecture which Ghidra and IDA both support.  See the source or product information of these tools:
https://github.com/NationalSecurityAgency/ghidra/tree/master/Ghidra/Processors
https://www.hex-rays.com/products/ida/processors.shtml

## Installation and Use

Currently, we officially support only Windows and Linux. It may be possible to build macOS version from the sources, but since I do not have a Mac, I cannot create a pre-built package, or continually make sure the macOS build is not broken.

1. Either download and unpack a pre-built package from the [latest release](https://github.com/chants/GhidraDec/releases/latest), or build and install the GhidraDec IDA plugin by yourself (the process is described below).

## Build and Installation

### Requirements

**Note: These are requirements to build the Ghidra IDA plugin, not to run it. See our user guide for information on plugin installation, configuration, and use.**

* A compiler supporting C++14
  * On Windows, only Microsoft Visual C++ is supported (version >= Visual Studio 2015).
* IDA SDK (version >= 7.0)

### Process

* Clone the repository:
  * `git clone https://github.com/chants/GhidraDec.git`
* Linux:
  * `cd ghidradec`
  * `mkdir build && cd build`
  * `cmake .. -DIDA_SDK_DIR=<path>`
  * `make`
  * `make install` (if `IDA_PATH` was set, see below)
* Windows:
  * Open a command prompt (e.g. `C:\msys64\msys2_shell.cmd` from [MSYS2])
  * if needed add to path such as SET PATH=%PATH%;%ProgramFiles%\cmake\bin;%UserProfile%\Documents\win_flex_bison
  * `cd GhidraDec`
  * `mkdir build && cd build`
  * cmake .. -DIDA_SDK_DIR=<path> -G<generator>
  * cmake --build . --config Release -- -m
  * cmake --build . --config Release --target install (if IDA_PATH was set, see below)
  * Alternatively, you can open `GhidraDec.sln` generated by `cmake` in Visual Studio IDE.

You must pass the following parameters to `cmake`:
* `-DIDA_SDK_DIR=</path/to/idasdk>` to tell `cmake` where the IDA SDK >= 7.0 directory is located.
* `-DIDA_SDK_DIR32=</path/to/idasdk>` to tell `cmake` where the IDA SDK < 7.0 directory is located.
* (Windows only) `-G<generator>` is `-G"Visual Studio 14 2015 Win64"` for 64-bit build using Visual Studio 2015. Later versions of Visual Studio may be used. Only 64-bit build is supported.

You can pass the following additional parameters to `cmake`:
* `-DIDA_PATH=</path/to/ida>` to tell `cmake` where to install the plugin. If specified, installation will copy plugin binaries into `IDA_PATH/plugins`, and content of `scripts/idc` directory into `IDA_PATH/idc`. If not set, installation step does nothing.

### IDA Installation and Configuration for Windows
The Windows version of the plugin requires Windows 7 or later, with the MSVC 2015 runtime installed.

1. Download the Windows installation package from the project’s release page.
2. Copy ghidradec.dll and ghidradec64.dll to the IDA’s plugin directory (<IDA_ROOT>/plugins).

### IDA Installation and Configuration for Linux
Follow the next steps to install RetDec plugin in a Linux environment:

1. Install 64-bit versions of the following shared-object dependencies:
libc.so.6 libgcc_s.so.1 libm.so.6 libpthread.so.0 libstdc++.so.6
2. Download the Linux installation package (Table 1) from the project’s release page.
3. Copy retdec.so and retdec64.so to the IDA’s plugin directory (<IDA_ROOT>/plugins).

### Depencency on Ghidra
It requires an extracted Ghidra release archive for the following files:
Ghidra/Processors/**
Ghidra/Features/Decompiler/os/win64/decompile.exe (on Windows 64)
Ghidra/Features/Decompiler/os/*/decompile (on Linux 64 or Mac 64)

### 3rd Party code listing
Statically linked are the following which for the .vcxproj must be manually fetched and extracted:

RetDec v3.3: https://github.com/avast/retdec config and utils libraries:
. -> retdec (requires at least include/retdec/config, include/retdec/utils, src/config, src/utils)

RetDec IDA Plugin v0.9: https://github.com/avast/retdec-idaplugin
src/idaplugin/*.cpp;*.h -> .

jsoncpp library v1.8.4: https://github.com/open-source-parsers/jsoncpp
. -> jsoncpp (at least src/lib_json, include/json)

whereami library: https://github.com/gpakosz/whereami (included via RetDec)
src/* -> whereami/

Ghidra decompiler and sleigh module: https://github.com/NationalSecurityAgency/ghidra
In Ghidra/Features/Decompiler/src/decompile/cpp/ -> decompile:
Following .hh/.cc/.y files for Sleigh: sleigh pcodeparse pcodecompile sleighbase slghsymbol slghpatexpress slghpattern semantics context filemanage
Following .hh/.cc/.y files for Core: xml space float address pcoderaw translate opcodes globalcontext
Following .hh/.cc files for LibSLA: loadimage memstate emulate opbehavior
Following .hh additional LibSLA files: types.h error.hh partmap.hh

pcodeparse.y and xml.y require bison
For Windows can use and make sure it is in the path: https://sourceforge.net/projects/winflexbison/ (at least win_bison.exe and the data folder)

### IDA’s plugin.cfg

The plugin’s default mode is set to selective decompilation. It tries to register hotkey CTRL+G for its invocation. If you already use this hotkey for another action or you just want to use a different hotkey, you need to modify IDA’s plugin configuration file. Moreover, the plugin supports one more decompilation mode and a hotkey invocation for the plugin’s configuration. If you want to use any of them, you also have to modify the config file. The IDA’s plugin configuration file is in <IDA_ROOT>/plugins/plugins.cfg. Its format is documented inside the file itself. To configure GhidraDec plugin, add the following lines at the beginning of the file:
; Plugin_name 						File_name 		Hotkey 		Arg
; -----------------------------------------------------------------
Ghidra_Decompiler 					ghidradec 		Ctrl-g 		 0
Ghidra_Decompiler_All 				ghidradec 		Ctrl-Shift-g 1
Ghidra_Decompiler_Configuration 	ghidradec 		Ctrl-Shift-c 2

These lines tell IDA which hotkeys invoke the plugin and what argument is passed to it. The plugin’s behavior after invocation is determined by the passed argument. Possible argument values are summarized in Table 3. In the provided example, we mapped selective decompilation to hotkey CTRL+D (plugin’s default), full decompilation to CTRL+SHIFT+D, and plugin configuration to CTRL+SHIFT+C. However, you may choose whichever hotkeys you like, provided they do not clash with other plugins or IDA.

Description of GhidraDec plugin’s invocation arguments.
Argument value 		Description
0 					Invokes selective decompilation.
1 					Invokes full decompilation.
2 					Invokes plugin configuration inside IDA.

## License

Copyright (c) 2019 chants, licensed under the MIT license. See the `LICENSE` file for more details.

GhidraDec IDA plugin uses third-party libraries or other resources listed, along with their licenses, in the `LICENSE-THIRD-PARTY` file.