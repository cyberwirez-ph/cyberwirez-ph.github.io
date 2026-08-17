---
title: "The PE File Format, Condensed: Structure, Headers, Imports & Relocations"
date: 2026-07-18
description: "Beginner-friendly notes covering the Windows PE file format: headers, sections, imports, and relocations."
author: "grit"
draft: false
---

*Originally published on [grit8086.github.io](https://grit8086.github.io/posts/pe-file-format/)*

Hello today we’re going to talk about PE file format, these are my notes while reading [0xrick](https://0xrick.github.io/win-internals/pe1/) ’s blog about PE file format. All credit goes to him, I only expanded my research & added visualizations and added explanations for the concepts and terminologies that isn’t so familiar to me.

PE stands for Portable Executable, a file format for executables used in Windows OS. It is based on the COFF file format. Dynamic link libraries (**.dll**), kernel modules (**.sys**), and control panel applications (**.cpl**) are also considered PE files.

Some quick history for context: Microsoft introduced PE with **Windows NT 3.1** , replacing the older 16-bit **NE** (New Executable) format. It was then adopted across Windows 95, 98, ME, and the Win32s extension for Windows 3.1x. It has picked up extensions since, notably **PE32+** for 64-bit address space and the .NET format for managed code. It’s also the executable format used by **UEFI** , so it reaches beyond Windows itself.

### What is COFF?

COFF is a standardized binary file format used to store compiled code and data before (or after) it gets turned into a final executable program. Think of it as a structured container that holds the output of a compiler, ready to be linked or loaded.

### Why not just compile to .EXE directly?

When you build a single source file in one shot, the compiler produces the **.obj** internally and the linker immediately consumes it, you just don’t see it happen. The problem shows up in multi-file projects: **program1.c** , **program2.c** , **program3.c** , etc. Every time you edit one line in **program2.c** , recompiling everything from scratch (even the files you didn’t touch) can take minutes to hours.

This is where compiling to COFF first helps. If you only changed **program2.c** , you recompile just that file to **program2.obj** and re-link. Two steps instead of a full rebuild, because internally **.c** files are turned into **.obj** files first.

The PE file format itself is a data structure that holds the information the OS loader needs to load an executable into memory and run it.


```
+------------------------------------------------+
|                  DOS HEADER                    |
+------------------------------------------------+
|                  DOS STUB                      |
|      (This program cannot be run in DOS)       |
+------------------------------------------------+
|                 NT HEADERS                     |
|   +----------------------------------------+   |
|   |        PE Signature (PE\0\0)           |   |
|   +----------------------------------------+   |
|   |            File Header                 |   |
|   +----------------------------------------+   |
|   |          Optional Header               |   |
|   +----------------------------------------+   |
+------------------------------------------------+
|                SECTION TABLE                   |
|         [ .text ][ .data ][ .bss ]...          |
+------------------------------------------------+
|             SECTION 1 (.text)                  |
|         // machine code lives here             |
+------------------------------------------------+
|             SECTION 2 (.data)                  |
|      // initialized global variables           |
+------------------------------------------------+
|             SECTION 3 (.bss)                   |
|       // uninitialized variables               |
+------------------------------------------------+
|             SECTION 4 (.rdata)                 |
|        // read-only data, strings              |
+------------------------------------------------+
|                     ...                        |
+------------------------------------------------+
|              SECTION N (...)                   |
|           // additional sections               |
+------------------------------------------------+
```


  * **DOS Header** — Every PE file starts with a 64-byte structure called the DOS Header, which makes the PE file an MS-DOS executable. It contains the magic number **MZ** (**0x5A4D**), which marks a valid DOS executable, and crucially, the **e_lfanew** field at offset **0x3C** , which points to the start of the NT Headers. This is what parsers use to locate the rest of the PE structure.

  * **DOS Stub** — A small MS-DOS 2.0 compatible executable that just prints an error message when the program is run in DOS mode. Dead code that nobody needs anymore.

  * **NT Headers** — Made up of three parts:

    * **PE Signature** — a 4-byte signature (**PE\0\0**) that identifies the file as a PE file.
    * **File Header** — the standard COFF file header. Contains basic information about the PE file such as target machine architecture, number of sections, and characteristics.
    * **Optional Header** — the most important header in the NT Headers. It provides critical information to the OS loader, including **ImageBase** , **EntryPoint** , and the **DataDirectories** array (which points to structures like the Import and Export tables). Its name is “Optional” because object files don’t have it, but it is required for image files like **.exe** and **.dll**.

> **What’s an image file?** In the context of PE files, an image file refers to a file that is meant to be loaded directly into memory and executed, like **.exe** and **.dll** files.

  * **Section Table** — An array of section headers, one per section in the PE file. To parse a specific section’s metadata, you iterate over this table.

  * **Sections** — Where the actual contents of the file are stored. Each section has its own purpose:

    * **.text** — compiled machine code
    * **.data** — initialized global variables
    * **.bss** — uninitialized variables. Takes up no space on disk; the OS loader simply zeroes out this region in memory at runtime.
    * **.rdata** — read-only data such as string literals and constants
    * **.idata** — import directory; describes which DLLs the executable depends on and which functions it imports from them. One of the most practically important sections for understanding how DLL loading works.



## DOS Header, DOS Stub, and Rich Header

The DOS header is a 64-byte structure at the very start of the PE file. It’s not functionally important anymore, it’s there for backward compatibility with MS-DOS. It makes the executable a valid MS-DOS program, so when loaded on MS-DOS, the DOS stub runs instead of the actual program.

### Structure


```c
// The DOS header is a legacy structure from the MS-DOS era.
// Every PE file (.exe, .dll, etc.) still starts with this header for backwards compatibility.
// Most fields here were relevant for MS-DOS executables and are essentially dead weight in modern PE files.
// The only two fields you actually care about as a PE parser are e_magic and e_lfanew.

typedef struct _IMAGE_DOS_HEADER {
    WORD   e_magic;                     // Must be 0x5A4D ("MZ"). Confirms the file is a valid DOS/PE executable.
    WORD   e_cblp;
    WORD   e_cp;
    WORD   e_crlc;
    WORD   e_cparhdr;
    WORD   e_minalloc;
    WORD   e_maxalloc;
    WORD   e_ss;
    WORD   e_sp;
    WORD   e_csum;
    WORD   e_ip;
    WORD   e_cs;
    WORD   e_lfarlc;
    WORD   e_ovno;
    WORD   e_res[4];
    WORD   e_oemid;
    WORD   e_oeminfo;
    WORD   e_res2[10];
    LONG   e_lfanew;                    // THE important field. File offset (bytes) pointing to the NT Headers (IMAGE_NT_HEADERS).
                                         // The OS loader jumps straight here to find the real PE structure.
                                         // Also the first thing any PE parser reads after validating e_magic.
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;

// IMAGE_DOS_HEADER is the struct type, used when you have a direct instance, e.g:
//   IMAGE_DOS_HEADER dosHeader;
//
// PIMAGE_DOS_HEADER is a pointer typedef to the same struct, used when you want to point to one, e.g:
//   PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)fileBase;
//
// This is the common Windows convention: P prefix = pointer. Saves you from writing IMAGE_DOS_HEADER* everywhere.
```


Only two members actually matter here:

  * **e_magic** — first member of the DOS header, a WORD (2 bytes). Serves as a signature that validates the file as an MS-DOS executable. Always **MZ** (**0x5A4D**).
  * **e_lfanew** — last member of the struct, located at offset **0x3C**. Holds an offset to the start of the NT Headers. This is important because it tells the PE loader where to find the actual PE structure.



### Rich Header

A chunk of data sitting between the DOS Stub and the NT Headers. It’s not an official part of the PE format, so you can zero it out completely and the executable still runs fine. One caveat if you’re going to try it: on a signed binary, that region falls inside the Authenticode hash, so zeroing it invalidates the signature.

It’s only present in executables built with Microsoft Visual Studio, and it stores metadata about the build tools used: tool type, version, and how many times each was used during compilation.

#### Structure

The data is XOR-encrypted. To read it, you XOR everything with the 32-bit checksum that follows the **Rich** signature at the end, which doubles as the XOR key.

Once decrypted, the layout is:


```
[ DanS ][ padding ][ entry ][ entry ][ entry ]...[ Rich ][ XOR key ]
```


  * **DanS** and **Rich** are magic signatures marking the start and end of the header
  * Each entry is a pair of DWORDs: one holding the tool type and build number, the other holding how many times that tool was used



#### Why should you care?

You probably won’t need to parse this yourself. What matters is knowing it exists and what it’s used for forensically.

The Rich Header is a fingerprint of the build environment. Malware analysts use it to attribute samples to threat actors, since two executables built on the same machine with the same toolset will have matching Rich Headers.

This is exactly what happened with **Olympic Destroyer** , malware used to disrupt the 2018 Winter Olympics. The authors copied the Rich Header from a known Lazarus Group sample into their own malware to fake attribution and throw analysts off.

## NT Headers

Before getting into NT Headers, let’s cover Image Base Address and Relative Virtual Address first.

### Image Base Address

The **ImageBase** is the memory address where the executable _prefers_ to be loaded, its starting point in memory. Every section’s location is expressed relative to this address (that’s what RVA is, see below).

It’s not guaranteed to actually be used though. Windows uses **ASLR** (Address Space Layout Randomization) as a security measure, so most executables get loaded at a random address instead of their preferred `ImageBase`. When that happens, the loader has to **relocate** the image, fixing up hardcoded addresses inside the binary to match wherever it actually landed. The **.reloc** section stores the list of spots that need fixing if this happens.

`ImageBase` must be a multiple of **64K**.

### Relative Virtual Address (RVA)

The RVA is the offset, in memory, where a specific part of the image begins once the program is loaded (relative to the Image Base). PE files rely heavily on RVAs.

Formula for converting between RVA and Virtual Address (VA):


```
RVA = Virtual Address - Image Base Address
VA  = Image Base Address + RVA
```


### NT Headers (IMAGE_NT_HEADERS)

**IMAGE_NT_HEADERS** is a struct in **winnt.h** that holds the core metadata of a PE file. It’s reached by following the pointer stored in the DOS header (**e_lfanew**).

Three members:

Member| Type| Purpose  
---|---|---  
Signature| `DWORD`| `"PE\0\0"` bytes, confirms valid PE  
FileHeader| `IMAGE_FILE_HEADER`| machine type, section count, timestamp, characteristics  
OptionalHeader| `IMAGE_OPTIONAL_HEADER`| entry point, image base, subsystem, data directories (imports/exports/resources/relocations)  
  
### 32-bit vs 64-bit versions


```
IMAGE_NT_HEADERS    -> PE32,  uses IMAGE_OPTIONAL_HEADER32
IMAGE_NT_HEADERS64  -> PE32+, uses IMAGE_OPTIONAL_HEADER64
```


**Signature** and **FileHeader** are identical in both. Only **OptionalHeader** differs.

### Why two versions exist

Comes down to pointer-sized fields. **ImageBase** and **SizeOfStackReserve** /**SizeOfStackCommit** need:

  * a full 64-bit address on PE32+
  * only a 32-bit address on PE32



So MS made two separate structs instead of one flexible one.

**What “pointer-sized” means:** the number of bytes needed to store a memory address. 32-bit systems use 4-byte addresses (max ~4GB range). 64-bit systems use 8-byte addresses (much bigger range). **ImageBase** is literally a memory address (where Windows loads the exe), so its field size has to match the bitness. Use a 4-byte field on a 64-bit binary and the address won’t fit, it gets truncated or corrupted.

### Practical implication (parsing PE files yourself)

You can’t know upfront which struct to cast to. Steps:

  1. Read the **Magic** field first, the first 2 bytes of the Optional Header (immediately after the File Header)
     * **0x10B** -> ( 32-bit, PE32 ) -> use **IMAGE_OPTIONAL_HEADER32**
     * **0x20B** -> ( 64-bit, PE32+ ) -> use **IMAGE_OPTIONAL_HEADER64**
  2. Only then cast/interpret **OptionalHeader** using the correct struct



Guess wrong here and every offset you read after this point will be misaligned.

Do **not** use `FileHeader.Machine` for this decision. It normally agrees with `Magic`, but the loader keys off `Magic`, and a crafted file can set the two inconsistently. See the Magic section below.

### FileHeader fields (IMAGE_FILE_HEADER)

  * **Machine** : number identifying the target CPU architecture. Many possible values, but the two you’ll actually see: **0x8664** for **AMD64** , **0x14c** for **i386**. Full list in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .
  * **NumberOfSections** : holds the number of section headers, which also gives you the size of the section table (each entry is a fixed 40 bytes, so `size = NumberOfSections × 40 bytes`).
  * **TimeDateStamp** : nominally a Unix timestamp of when the file was created. Don’t trust it. MSVC’s `/Brepro` switch writes a hash here instead of a real time for reproducible builds, and the field is trivially forged anyway.
  * **PointerToSymbolTable** / **NumberOfSymbols** : file offset to the COFF symbol table and its entry count. Always **0** in practice. The COFF symbol table tracked named things in object files (function names, global variables) and their offsets, used by the linker to resolve function calls across files. It’s deprecated now, modern PE files use separate debug formats (like PDB) instead, so these fields are just legacy leftovers.
  * **SizeOfOptionalHeader** : size of the Optional Header, in bytes. In practice **0xE0** for PE32 and **0xF0** for PE32+, which makes it a useful sanity check while parsing.
  * **Characteristics** : flags describing file attributes, executable, system file vs user program, etc. Full list in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .



## Optional Header (IMAGE_OPTIONAL_HEADER)

The most important header in the NT Headers. This is where the PE loader gets the specifics it needs to actually load and run the executable.

Called “optional” because some file types (like object files, **.obj**) don’t have it. But for image files (**.exe** , **.dll**), it’s required.

It has no fixed size, that’s why **SizeOfOptionalHeader** exists in the FileHeader, it tells parsers how many bytes to read for this header.

The header opens with the standard COFF fields (part of the original COFF spec): 9 members / 28 bytes on PE32, 8 members on PE32+, since **BaseOfData** only exists in the 32-bit version. Everything after that group is a Microsoft extension, added specifically for the Windows PE loader and linker.

### 32-bit vs 64-bit, the exact differences

Two things differ between **IMAGE_OPTIONAL_HEADER32** and **IMAGE_OPTIONAL_HEADER64** :

  1. **Member count** : 32-bit version has 31 members, 64-bit has 30. The extra one in the 32-bit version is **BaseOfData** (a DWORD holding the RVA of the start of the data section). It doesn’t exist in the 64-bit struct at all.
  2. **Field size** : these 5 members are **DWORD** (4 bytes) in the 32-bit version and **ULONGLONG** (8 bytes) in the 64-bit version:
     * **ImageBase**
     * **SizeOfStackReserve**
     * **SizeOfStackCommit**
     * **SizeOfHeapReserve**
     * **SizeOfHeapCommit**



### Magic: what actually decides 32-bit vs 64-bit

Important gotcha: it’s **not** `FileHeader.Machine` that the Windows PE loader checks to decide bitness. It’s the **Magic** field, right here in the Optional Header.

Values:

  * **0x10B** -> PE32 (32-bit)
  * **0x20B** -> PE32+ (64-bit)
  * **0x107** -> ROM image



### Fields

**Standard COFF fields:**

  * **Magic** : determines PE32 vs PE32+ vs ROM image (see above). This is the field that actually decides bitness, not `Machine`.
  * **MajorLinkerVersion** / **MinorLinkerVersion** : version number of the linker used to build the file.
  * **SizeOfCode** : size of the **.text** section, or the sum of all code sections if there are multiple.
  * **SizeOfInitializedData** : size of the **.data** section, or the sum of all initialized data sections.
  * **SizeOfUninitializedData** : size of the **.bss** section, or the sum of all uninitialized data sections.
  * **AddressOfEntryPoint** : RVA of the entry point once loaded into memory. For normal executables this is the actual starting address. For device drivers it points to the init function (usually `DriverEntry`). Optional for DLLs, set to **0** if the DLL has no entry point.
  * **BaseOfCode** : RVA of the start of the code section once loaded into memory.
  * **BaseOfData** (32-bit only): RVA of the start of the data section once loaded into memory.



**Windows-specific (NT additional) fields:**

  * **ImageBase** : preferred address to load the image at (must be a multiple of 64K). Typical defaults are **0x400000** for x86 executables, **0x140000000** for x64 executables, and **0x10000000** for DLLs. Rarely actually used due to ASLR and other memory protections, in that case the loader picks a different address and **relocates** the image, fixing up hardcoded addresses to match. The section holding relocation info is **.reloc**.
  * **SectionAlignment** : alignment boundary (bytes) for sections in memory. Defaults to the architecture’s page size. Can’t be smaller than **FileAlignment**.
  * **FileAlignment** : alignment boundary (bytes) for section raw data on disk. Any leftover space gets zero-padded. Must be a power of 2 between 512 and 64K. If **SectionAlignment** is smaller than the page size, this must match **SectionAlignment** exactly.
  * **MajorOperatingSystemVersion** / **MinorOperatingSystemVersion** , **MajorImageVersion** / **MinorImageVersion** , **MajorSubsystemVersion** / **MinorSubsystemVersion** : version numbers for the required OS, the image itself, and the required subsystem, respectively.
  * **Win32VersionValue** : reserved, should be **0**.
  * **SizeOfImage** : total size of the image (bytes), including all headers. Rounded up to a multiple of **SectionAlignment** , used when loading into memory.
  * **SizeOfHeaders** : combined size of the DOS stub, NT Headers, and section headers. Rounded up to a multiple of **FileAlignment**.
  * **CheckSum** : checksum of the image. It is only actually validated for kernel-mode drivers, boot-time DLLs, and certain critical system images. Ordinary user-mode executables load fine with a wrong or zero checksum, which is why so many packed samples have one.
  * **Subsystem** : which Windows subsystem the image needs to run (GUI, console, etc). Full list in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .
  * **DllCharacteristics** : flags describing image characteristics, e.g. NX-compatible, relocatable at runtime. Despite the name, applies to normal executables too, not just DLLs. Full list in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .
  * **SizeOfStackReserve** / **SizeOfStackCommit** / **SizeOfHeapReserve** / **SizeOfHeapCommit** : how much stack/heap space to reserve vs actually commit upfront.
  * **LoaderFlags** : reserved, should be **0**.
  * **NumberOfRvaAndSizes** : size of the **DataDirectory** array.
  * **DataDirectory** : array of `IMAGE_DATA_DIRECTORY` entries, pointers to structures like the Import Table, Export Table, and Relocation Table.



## Alignment and memory permissions

This expands on why **SectionAlignment** exists. The short version: without it, a single page in memory could be forced to hold two sections that need different permissions, and the OS can’t split permissions within one page.

### Without alignment (sections not page-aligned)


```
Address    Memory Layout                          Page Permission
-------    ------------------------------------   ------------------------
0x1000     +----------------------------------+
           | .text (needs: R-X)               |   Page 1 (0x1000-0x1FFF)
0x18F0     |----------------------------------|   <- .text ends mid-page
           | .data (needs: RW-)               |   <- .data starts on the SAME page
0x2000     +----------------------------------+
           | .data continued (RW-)            |   Page 2 (0x2000-0x2FFF)
           +----------------------------------+
```


Page 1 now has to satisfy two conflicting requirements at once: executable (for **.text**) and writable (for **.data**). Since permissions apply per whole page, not per byte, the OS only has bad options here:

  * mark the whole page **RWX** (writable + executable at the same time), which is a serious security hole, or
  * get the permissions wrong for whichever section didn’t “win”



Either way, the memory protection model breaks.

### With alignment (SectionAlignment = 0x1000 / 4096 bytes)


```
Address    Memory Layout                          Page Permission
-------    ------------------------------------   -------------------------
0x1000     +----------------------------------+
           | .text        (R-X)               |   Page 1 = R-X only
0x18F0     | unused tail of .text's last page |
0x2000     +----------------------------------+
           | .data        (RW-)               |   Page 2 = RW- only
0x2410     | unused tail of .data's last page  |
0x3000     +----------------------------------+
           | .rdata       (R--)               |   Page 3 = R-- only
0x3220     | unused tail of .rdata's last page |
0x4000     +----------------------------------+
```


Every section now starts at a fresh page boundary. Each page holds exactly one section, so it only ever needs one permission. No conflicts, no forced RWX pages.

The padding is the unused remainder of each section’s **final** page, not a whole page inserted between sections. A section bigger than 4K simply spans as many pages as it needs, and the next section starts at the boundary after that. So the wasted space per section is at most one page minus a byte, which is the cost of buying this clean separation.

The permissions themselves aren’t inferred from the section name. They come from the `Characteristics` bitmask in each section header, covered in Section Headers below.

## Data Directories and Sections

### Data Directories

The last member of **IMAGE_OPTIONAL_HEADER** is the **DataDirectory** array, up to **16** entries (`IMAGE_NUMBEROF_DIRECTORY_ENTRIES`). Each entry is a simple struct:


```c
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;   // RVA to the start of the data directory
    DWORD   Size;             // size of the data directory
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```


A **Data Directory** is just a chunk of data sitting inside one of the PE file’s sections. Not all of them have the same internal structure, the **type** of directory determines how its data gets parsed. If an entry has both `VirtualAddress` and `Size` set to **0** , that directory doesn’t exist in this file.

The 16 directory slots (index into the `DataDirectory` array):

Index| Directory  
---|---  
0| Export Directory  
1| Import Directory  
2| Resource Directory  
3| Exception Directory  
4| Security Directory  
5| Base Relocation Table  
6| Debug Directory  
7| Architecture Specific  
8| RVA of GP  
9| TLS Directory  
10| Load Config Directory  
11| Bound Import Directory  
12| Import Address Table  
13| Delay Load Imports  
14| COM Runtime Descriptor  
15| Reserved (must be zero)  
  
> **One exception to the RVA rule:** index 4, the Security Directory (Certificate Table, where the Authenticode signature lives). Its `VirtualAddress` is a **raw file offset** , not an RVA, because the certificate data is not mapped into memory at all. Every other directory uses an RVA. This is the single most common bug in hand-written PE parsers.

The **Import Directory** (index 1) is one of the most important ones, it lists every external function the executable pulls in from other DLLs.

### Sections

Sections hold the actual content of the file, they sit after the headers and section headers, taking up the rest of the file. Special section names and their purpose:

  * **.text** — executable code
  * **.data** — initialized data
  * **.bss** — uninitialized data
  * **.rdata** — read-only initialized data
  * **.edata** — export tables
  * **.idata** — import tables
  * **.reloc** — image relocation info
  * **.rsrc** — resources (images, icons, embedded binaries)
  * **.tls** — Thread Local Storage (see below)
  * **.pdata** — exception handling data (see below)



Full list of special section names in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .

> **Names are conventions, not guarantees.** Modern MSVC merges import data into **.rdata** , so **.idata** frequently does not exist in a real binary. It generally doesn’t emit **.bss** either, uninitialized data goes into **.data** with `VirtualSize` larger than `SizeOfRawData`. Never locate anything by section name. The Data Directories are authoritative, follow the RVA and work out which section contains it.

#### What is TLS (Thread Local Storage)?

Not the network security protocol, unrelated. **TLS here means each thread gets its own private copy of a variable** , instead of one global copy shared (and fought over) by every thread.

Useful when multiple threads need “the same” variable conceptually, but each thread’s value should stay isolated (e.g. a per-thread “last error” value). Without this, threads would overwrite each other’s data, race conditions, corruption.

The **.tls** section doesn’t hold the actual per-thread runtime data itself (that’s created fresh per thread at runtime). It holds the **setup info** : initial values used as a template for each thread’s copy, and callback functions that run when a thread starts or ends.

> **Security note:** malware sometimes hides code inside a **TLS callback** , because those callbacks run _before_ the normal entry point (`AddressOfEntryPoint`). An analyst who only checks the entry point can miss code that already ran via a TLS callback.

#### .pdata and the Exception Directory

Data directory index 3, the **Exception Directory** , points into the **.pdata** section. This is not optional detail on 64-bit builds. x64 and ARM64 use **table-based exception handling** , meaning the unwind information lives in a static table in the file rather than being built on the stack at runtime the way 32-bit x86 SEH does. So **.pdata** is present in essentially every x64 binary and absent from most x86 ones.

The section holds an array of `RUNTIME_FUNCTION` structs, one per non-leaf function in **.text** :

  * **BeginAddress** / **EndAddress** : RVAs bounding the function.
  * **UnwindInfoAddress** : RVA of that function’s `UNWIND_INFO`, which describes how to restore the stack and registers while unwinding, and points to an exception handler if the function has one.



A leaf function (one that calls nothing and touches no non-volatile registers) needs no entry.

Practically this matters twice over. It’s a rough function map of the binary, which is why disassemblers use it to recover function boundaries. And if you’re writing code that gets mapped manually rather than by the loader, missing or wrong **.pdata** means exceptions inside your code have no unwind data and the process dies instead of handling them.

### Section Headers

Sit between the Optional Header and the actual sections. One `IMAGE_SECTION_HEADER` entry per section, describing that section’s metadata:


```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;
    } Misc;
    DWORD   VirtualAddress;
    DWORD   SizeOfRawData;
    DWORD   PointerToRawData;
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```


  * **Name** : section name, max **8 characters** (`IMAGE_SIZEOF_SHORT_NAME = 8`). Longer names would normally use a string table offset workaround, but executable images don’t use a string table, so 8 chars is a hard limit here.
  * **Misc (PhysicalAddress / VirtualSize)** : a union, both names refer to the same field. Holds the section’s total size once loaded in memory.
  * **VirtualAddress** : for executable images, the RVA of the section’s first byte once loaded (relative to Image Base). For object files, the address before relocation is applied.
  * **SizeOfRawData** : size of the section on disk. Must be a multiple of **FileAlignment**.
  * **PointerToRawData** : file offset to the section’s first page. Must be a multiple of **FileAlignment** for executable images.
  * **PointerToRelocations** / **NumberOfRelocations** : file pointer and count of relocation entries. Both **0** for executable files.
  * **PointerToLinenumbers** / **NumberOfLinenumbers** : file pointer and count of COFF line-number entries. Both **0** , COFF debugging info is deprecated.
  * **Characteristics** : flags describing the section, executable code, initialized/uninitialized data, shareable in memory, etc. This is what marks a section read-only, read-write, or read+execute. The three that decide the page permissions discussed in the alignment section :
```
0x20000000   IMAGE_SCN_MEM_EXECUTE
0x40000000   IMAGE_SCN_MEM_READ
0x80000000   IMAGE_SCN_MEM_WRITE
```
So a normal **.text** is `0x60000000` (read + execute), **.data** is `0xC0000000` (read + write), and **.rdata** is `0x40000000` (read only). A section carrying all three, `0xE0000000`, is RWX, which is the thing the whole alignment design exists to avoid. Full list in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .



#### Why SizeOfRawData and VirtualSize can differ

Case| Cause| Bigger one  
---|---|---  
Disk padding| SizeOfRawData gets rounded up to a multiple of FileAlignment, but VirtualSize reflects the real, unpadded size| SizeOfRawData  
Uninitialized data| Data with no value yet (e.g. .bss) isn’t stored on disk at all, but memory still reserves space for it since it gets used at runtime| VirtualSize  
  
Tools like PE-bear show friendlier labels for these fields:

PE-bear label| Actual struct field  
---|---  
Raw Addr.| `PointerToRawData`  
Virtual Addr.| `VirtualAddress`  
Raw Size| `SizeOfRawData`  
Virtual Size| `VirtualSize`  
  
To find where a section ends, just add start + size:


```
On disk:    Raw Addr (0x400)      + Raw Size (0xE00)      = 0x1200
In memory:  Virtual Addr (0x1000) + Virtual Size (0xD2C)  = 0x1D2C
```


Different raw size vs virtual size here is exactly the disk-padding / uninitialized-data behavior described above.

## PE Imports

Covers how an executable pulls in functions it doesn’t define itself, from other DLLs. Import data traditionally lives in the **.idata** section, though modern MSVC folds it into **.rdata** , so locate it through data directory entry 1 rather than by section name. Either way it’s tracked through the same chain of structures: an entry per DLL, a list of needed functions, and a table of resolved addresses.

### Import Directory Table

Located by data directory entry 1, traditionally at the start of **.idata**. An array of **IMAGE_IMPORT_DESCRIPTOR** structs, one entry per imported DLL. No fixed size, the array ends with a zeroed-out entry.


```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;
        DWORD   OriginalFirstThunk;
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp;
    DWORD   ForwarderChain;
    DWORD   Name;
    DWORD   FirstThunk;
} IMAGE_IMPORT_DESCRIPTOR;
```


  * **OriginalFirstThunk** : RVA of the ILT for this DLL.
  * **TimeDateStamp** : **0** if this import is unbound. **-1** if bound (see Bound Imports below for what that means).
  * **ForwarderChain** : index of the first forwarder chain reference. Related to **DLL forwarding** , when a DLL forwards some of its exported functions to another DLL.
  * **Name** : RVA of an ASCII string, the name of the imported DLL.
  * **FirstThunk** : RVA of the IAT for this DLL.



### Import Lookup Table (ILT)

Also called the **Import Name Table (INT)**. Every imported DLL has one, pointed to by `OriginalFirstThunk`. It’s a list telling the loader exactly which functions are needed from that DLL.

Structurally: an array of 32-bit numbers (PE32) or 64-bit numbers (PE32+). Last entry is zeroed-out to mark the end.

Each entry encodes info using its bits, not separate fields:

  * **Most significant bit** (bit 31 for PE32, bit 63 for PE32+): the **Ordinal/Name flag**. Decides how to read the rest of the number.
  * If flag = **1** (import by ordinal): bits 15-0 hold the 16-bit ordinal number directly. The bits in between must be **0**.
  * If flag = **0** (import by name): bits 30-0 hold an RVA pointing to a Hint/Name table.


```
Flag = 1 (by ordinal), 32-bit entry:
Bit:    31            30 ... 16   15 ...........0
       [ 1 ]          [  0000...  ]  [ordinal number]

Flag = 0 (by name), 32-bit entry:
Bit:    31                    30 ....................... 0
       [ 0 ]                 [        RVA value          ]
```


**Import by name** vs **import by ordinal** : by name identifies a function using its actual string name (e.g. `"CreateFileA"`), the loader matches text against the DLL’s export table. By ordinal identifies a function using just a number, an index into the DLL’s export table, no string comparison needed. Ordinal is faster but fragile: if the DLL’s exports get reordered in a future version, the same ordinal number can point to a different function. Name-based imports don’t have that risk since the name itself doesn’t change.

### Hint/Name Table

What a by-name ILT entry’s RVA points to. Defined as `IMAGE_IMPORT_BY_NAME`:


```c
typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD    Hint;
    CHAR    Name[1];
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```


  * **Hint** : a number used as a shortcut, first tried as a direct index into the DLL’s export name pointer table. If that guess is wrong, falls back to a binary search over the export table.
  * **Name** : null-terminated string, the actual function name to import (e.g. `"CreateFileA"`).



### Import Address Table (IAT)

Pointed to by `FirstThunk`. On disk, the IAT is **identical** to the ILT, same entries, same encoding. The difference only shows up at runtime: once the executable is loaded, the loader resolves each function’s real address and **overwrites the IAT entries** with those addresses. The ILT itself stays untouched, it’s the reference copy; the IAT is the one that gets rewritten with real addresses and is what the compiled code actually calls into.

That assumes both tables are present, which is typical MSVC output but not required by the format. `OriginalFirstThunk` can be **0** , in which case there is no ILT at all and only the IAT exists. Any parser you write has to handle that case.

> **Malware relevance** : **IAT hooking** overwrites IAT entries with addresses pointing to attacker-controlled code, intercepting calls to legitimate functions. This targets the IAT specifically, never the ILT, because the IAT is what execution actually reads from.

#### ILT vs IAT, quick recap

Aspect| ILT| IAT  
---|---|---  
Pointed to by| `OriginalFirstThunk`| `FirstThunk`  
Content (disk)| ordinal or RVA to name| identical to ILT  
Content (runtime)| unchanged, stays the same| overwritten w/ real addrs  
Role| reference / lookup list| what code actually calls  
Gets modified?| no| yes, by the loader  
  
On disk they’re identical. At runtime, the ILT stays fixed (a stable reference of what was requested), while the IAT gets overwritten with resolved addresses, it’s the table the compiled code’s `call` instructions actually read from. Having two separate tables means the original name/ordinal list survives even after addresses get filled in, useful for analysis, debugging, or verifying a binary hasn’t been tampered with post-load.

### Bound Imports

A **bound import** means the IAT already contains fixed, precomputed function addresses, written into the file ahead of time instead of being resolved by the loader at runtime.

Binding is not something the compiler does. It’s a post-link step, run by **bind.exe** or the linker’s **/BIND** switch, against the DLLs actually present on the machine at the time it runs.

The idea was speed: skip having the loader resolve addresses and fill the IAT at load time. The catch: if the real addresses at runtime don’t match what was precomputed (target DLL updated, or relocated somewhere else), the loader falls back to resolving them again and fixing the IAT anyway, so the gain was never guaranteed.

Worth knowing that this is effectively dead in practice. From Vista onward, ASLR means system DLLs almost never land at the address they were bound against, so the fallback path is the normal path. You will rarely see bound imports in a modern binary.

This is exactly what `TimeDateStamp = -1` in `IMAGE_IMPORT_DESCRIPTOR` signals: “this import is bound.” The real timestamp of the bound DLL isn’t stored there, it’s found instead in the Bound Import Data Directory.

### Bound Import Data Directory

A separate Data Directory holding info specifically about bound imports. Structurally similar to the Import Directory Table: an array of `IMAGE_BOUND_IMPORT_DESCRIPTOR` structs, ending in a zeroed-out entry.


```c
typedef struct _IMAGE_BOUND_IMPORT_DESCRIPTOR {
    DWORD   TimeDateStamp;
    WORD    OffsetModuleName;
    WORD    NumberOfModuleForwarderRefs;
    // Array of zero or more IMAGE_BOUND_FORWARDER_REF follows
} IMAGE_BOUND_IMPORT_DESCRIPTOR, *PIMAGE_BOUND_IMPORT_DESCRIPTOR;
```


  * **TimeDateStamp** : the real timestamp of the bound DLL.
  * **OffsetModuleName** : offset (from the first `IMAGE_BOUND_IMPORT_DESCRIPTOR`) to a string with the DLL’s name.
  * **NumberOfModuleForwarderRefs** : how many `IMAGE_BOUND_FORWARDER_REF` structs immediately follow this one. `IMAGE_BOUND_FORWARDER_REF` is identical to `IMAGE_BOUND_IMPORT_DESCRIPTOR`, except its last member is reserved.



### How it all connects


```
IMAGE_IMPORT_DESCRIPTOR  (one of these per imported DLL)
+--------------------------------------------------+
| OriginalFirstThunk  ----+                        |
| TimeDateStamp           |                        |
| ForwarderChain          |                        |
| Name  ------------------|-----+                  |
| FirstThunk  -------------------|-----+           |
+--------------------------------------------------+
                          |     |      |
                          |     |      |
                          v     |      v
                  ILT (array)   |    IAT (array)
              +---------------+ |  +----------------------+
              | entry 0       | |  | entry 0  (real addr) | <- overwritten
              | entry 1       | |  | entry 1  (real addr) |    at load time
              | ...           | |  | ...                  |
              | 0x0 (end)     | |  | 0x0 (end)            |
              +---------------+ |  +----------------------+
                    |           |
                    |           v
                    |     "kernel32.dll"
                    |     (DLL name string)
                    |
         each entry's top bit decides:
         +----------------------------+   +------------------------------+
         | flag = 1  (by ordinal)     |   | flag = 0  (by name)          |
         | bits 15-0 = ordinal number |   | bits 30-0 = RVA to Hint/Name |
         +----------------------------+   +--------------+---------------+
                                                          |
                                                          v
                                             IMAGE_IMPORT_BY_NAME
                                             +------------------------+
                                             | Hint                   |
                                             | Name (e.g. CreateFileA)|
                                             +------------------------+
```


Reading it top to bottom: one **IMAGE_IMPORT_DESCRIPTOR** per DLL, which points to that DLL’s name, its ILT, and its IAT. Before the image is loaded, the ILT and IAT are identical, every entry either an ordinal number or an RVA to a Hint/Name struct. After loading, the IAT entries get overwritten with real addresses, while the ILT stays as the original reference list.

For every DLL an executable imports from:

  1. There’s an **IMAGE_IMPORT_DESCRIPTOR** in the Import Directory Table, holding the DLL name and RVAs to that DLL’s ILT and IAT.
  2. The ILT lists every function needed from that DLL (by name or ordinal).
  3. The IAT starts out identical to the ILT, then gets overwritten with real function addresses once the executable is loaded.
  4. If the import is bound, addresses are precomputed ahead of time by a post-link binding step instead, and the real DLL timestamp lives in a separate Bound Import Data Directory.



One thing this doesn’t cover: **delay-load imports** (data directory index 13, built with the linker’s `/DELAYLOAD` switch). Those work differently enough to deserve their own treatment. The DLL isn’t loaded at process start at all, and the first call to a delay-loaded function goes through a helper stub that loads the DLL and resolves the address on the spot, rather than through the loader. Worth knowing it exists, since a binary’s real dependency list isn’t just what’s in the normal Import Directory.

## PE Base Relocations

### Relocations, the concept

At compile time, the compiler assumes the executable will load at a specific base address (**ImageBase**). Based on that assumption, some addresses get calculated and **hardcoded** directly into the binary.

At runtime, the executable rarely actually lands at that assumed address (ASLR, address already taken, etc). Once that happens, every one of those hardcoded addresses is now wrong.

The fix: a list of every hardcoded location that would need adjusting if the load address differs, stored in the **Relocation Table** (a Data Directory inside the **.reloc** section). The loader walks this list and patches each one, this process is called **relocating**.

#### Worked example


```c
int test = 2;
int* testPtr = &test;
```


  * Compile time: assumed **ImageBase = 0x400000**. Compiler places **test** at offset **0x100** , so `test`’s address = `0x400100`. `testPtr` gets hardcoded to `0x400100`.
  * Runtime: image actually loads at `0x500000` instead.
  * `testPtr` is now stale, still holds `0x400100`, which no longer points to `test`.
  * Loader computes the difference between actual and assumed base:
```
difference = actual_base - assumed_base = 0x500000 - 0x400000 = 0x100000
```

  * Adds that difference to every flagged address, including `testPtr`:
```
new testPtr = old testPtr + difference = 0x400100 + 0x100000 = 0x500100
```




**0x500100** is exactly where **test** actually ended up after loading at the new base, so the pointer is valid again.

A list is needed instead of just scanning the whole binary because not every number in a binary is an address, plenty of it is plain data (constants, string bytes, etc) that shouldn’t be touched. The Relocation Table tells the loader precisely which offsets are addresses that need this fix.

### Relocation Table structure

Lives in the **.reloc** section, split into **blocks**. Each block covers the relocations for one **4K page** , and each block must start on a 32-bit boundary.

Every block starts with an **IMAGE_BASE_RELOCATION** struct, followed by however many offset entries that block needs:


```c
typedef struct _IMAGE_BASE_RELOCATION {
    DWORD   VirtualAddress;
    DWORD   SizeOfBlock;
} IMAGE_BASE_RELOCATION, *PIMAGE_BASE_RELOCATION;
```


  * **VirtualAddress** : RVA of the page this block covers.
  * **SizeOfBlock** : total size (bytes) of this block, header + all its offset entries.



#### Offset entries

Each entry is a single **WORD** (2 bytes), split into two parts:

  * **First 4 bits** : relocation type. Full list in the [official Microsoft documentation](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) .
  * **Last 12 bits** : offset from the page’s **VirtualAddress** (the RVA in the block header).


```
WORD entry (16 bits):
Bit:   15 12  11                    0
      [type][      offset          ]
```


One type is worth calling out: type **0** , `IMAGE_REL_BASED_ABSOLUTE`, is a no-op. It’s used purely as padding to keep blocks 32-bit aligned, and must be skipped rather than patched.

To get the actual location that needs fixing: **location = actual load base + page RVA + offset**.

Note that it’s the base the image _actually_ loaded at, not `ImageBase`. If relocation is happening at all, the image is not sitting at `ImageBase`, that’s the whole reason the loader is walking this table.

#### Example (one relocation block)

A PE file with a single relocation block, `SizeOfBlock = 0x28` (40 bytes).

  * Block header (**IMAGE_BASE_RELOCATION**) is always 8 bytes.
  * Remaining space for entries: **0x28 - 8 = 0x20** (32 bytes).
  * Each entry is 2 bytes, so: **0x20 / 2 = 16** entries in this block.



16 _entries_ , though, not necessarily 16 relocations. Any of them with type 0 are alignment padding and get skipped, so the real count of patched addresses can be lower.

## Static detection opportunities

Everything above is format mechanics. This section is why the mechanics matter defensively: AV engines, static analysis pipelines, and EDR sensors parse exactly these structures to score a file _before_ it ever executes. What they’re looking for is deviation from what a normal compiler produces.

Nothing below is an indicator of compromise on its own. Every single one has legitimate causes. They matter in combination.

### Whole-file indicators

  * **Hashes and imphash** : MD5/SHA256 get checked against threat intel feeds, but those change with a single recompile. **Imphash** is more durable: a hash over the ordered list of DLL and function names from the import descriptors. Two samples from the same family, or built from the same source with minor changes, often share an imphash even when their file hashes differ. Note that it hashes the _names_ , not the resolved addresses, since those change every load.
  * **Section entropy** : Shannon entropy per section, on a 0 to 8 scale. Above roughly 7.0 means the data is close to random, which in practice means compressed, encrypted, or packed. A high-entropy **.text** is unusual since normal machine code sits lower.
  * **Authenticode signature** : valid, absent, or signed with a stolen or revoked certificate. Recall from the Rich Header section that the signature covers most of the file, so tampering breaks it.



### Header and section table indicators

From `IMAGE_FILE_HEADER`:

  * **TimeDateStamp** : checked for **time stomping**. A compile date in the future, or one cloned to match a legitimate system binary exactly, is anomalous. Remember this field is also legitimately non-temporal under `/Brepro`, which is why it’s weak evidence alone.
  * **NumberOfSections** : unusually few (1 or 2) or unusually many is a deviation from normal compiler output.



From `IMAGE_OPTIONAL_HEADER`:

  * **AddressOfEntryPoint** : if the entry point RVA falls outside **.text** , or inside a section appended at the end of the file, that’s a classic packer or file-infector shape. A stub runs first, unpacks, then jumps to the real code.
  * **SizeOfImage** vs actual file size: a large amount of data past the end of the last section (an **overlay**) sits outside the defined PE structure entirely and is a common place to hide a second-stage payload.
  * **Subsystem** : a GUI-subsystem binary that never creates a window runs with no visible presence.



From the section table:

  * **RWX sections** (`Characteristics = 0xE0000000`): rare in legitimate modern software because of DEP/NX. Implies self-modifying code or in-memory unpacking.
  * **VirtualSize far exceeding SizeOfRawData** : some gap is normal, per the earlier section . A raw size of zero with a large virtual size means space is being reserved for something written at runtime, which is what an unpacking stub needs.
  * **Section names** : randomized or tool-specific names (`.upx0`, `.vmp0`, alphanumeric noise) effectively self-identify the packer. Legitimate binaries mostly stick to the conventional set.



### Import table indicators

The Import Directory is the richest static signal available, because it declares what the binary intends to do with the OS.

  * **API groupings** : engines look for combinations tied to a technique rather than individual calls. `VirtualAllocEx` \+ `WriteProcessMemory` \+ `CreateRemoteThread` is process injection. `SetWindowsHookEx` suggests keylogging.
  * **Sparse imports** : an import table containing little more than `LoadLibrary` and `GetProcAddress` means the real imports are resolved at runtime, often via hashed API names, specifically to blind static analysis. Dynamic resolution by itself is entirely normal. Resolving _everything_ that way, with a near-empty IAT, is the anomaly.
  * **Exports** (for DLLs): random or meaningless export names, or exporting only by ordinal with no names at all, removes information an analyst would otherwise get for free.
  * **Resources** : **.rsrc** holds arbitrary data by design, so it gets parsed for nested PE files, scripts, and high-entropy blobs, and the application manifest gets checked for privilege requests like `requireAdministrator`.



The through-line: a packer or loader has to leave structural traces, because the format demands consistency that the packing process breaks. Understanding the format is what lets you see which traces are unavoidable and which are just careless.

## Credits

  * [0xrick - PE file format](https://0xrick.github.io/win-internals/pe1/)


