# Object Files

A Object File is a file, that contains compilation output (object code). It is sort of the "base compilation output file".

An Object File is the output of compiling a compilation unit, which is effectively the code to be compiled of a single source code file. It in of itself isn't really usable, either as an executable or as a shared library (note how below, the entry point address is `0x00`). However object files can be linked together, to create either executables or shared libraries.

These files typically have the `.o` file extension.

Now since object files are only made to be linked not executed, it doesn't have a program header table, only a section header table. Which we see here:

```
$	readelf -a test.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          912 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         14
  Section header string table index: 13

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       000000000000004e  0000000000000000  AX       0     0     1
  [ 2] .rela.text        RELA             0000000000000000  00000240
       0000000000000090  0000000000000018   I      11     1     8
  [ 3] .data             PROGBITS         0000000000000000  0000008e
       0000000000000000  0000000000000000  WA       0     0     1
  [ 4] .bss              NOBITS           0000000000000000  0000008e
       0000000000000000  0000000000000000  WA       0     0     1
  [ 5] .rodata           PROGBITS         0000000000000000  0000008e
       000000000000000c  0000000000000000   A       0     0     1
  [ 6] .comment          PROGBITS         0000000000000000  0000009a
       000000000000002c  0000000000000001  MS       0     0     1
  [ 7] .note.GNU-stack   PROGBITS         0000000000000000  000000c6
       0000000000000000  0000000000000000           0     0     1
  [ 8] .note.gnu.pr[...] NOTE             0000000000000000  000000c8
       0000000000000020  0000000000000000   A       0     0     8
  [ 9] .eh_frame         PROGBITS         0000000000000000  000000e8
       0000000000000078  0000000000000000   A       0     0     8
  [10] .rela.eh_frame    RELA             0000000000000000  000002d0
       0000000000000048  0000000000000018   I      11     9     8
  [11] .symtab           SYMTAB           0000000000000000  00000160
       00000000000000c0  0000000000000018          12     4     8
  [12] .strtab           STRTAB           0000000000000000  00000220
       0000000000000019  0000000000000000           0     0     1
  [13] .shstrtab         STRTAB           0000000000000000  00000318
       0000000000000074  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)

There are no section groups in this file.

There are no program headers in this file.

There is no dynamic section in this file.

Relocation section '.rela.text' at offset 0x240 contains 6 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
00000000000b  000300000002 R_X86_64_PC32     0000000000000000 .rodata - 4
000000000013  000500000004 R_X86_64_PLT32    0000000000000000 puts - 4
000000000025  000300000002 R_X86_64_PC32     0000000000000000 .rodata + 0
00000000002d  000500000004 R_X86_64_PLT32    0000000000000000 puts - 4
00000000003f  000300000002 R_X86_64_PC32     0000000000000000 .rodata + 4
000000000047  000500000004 R_X86_64_PLT32    0000000000000000 puts - 4

Relocation section '.rela.eh_frame' at offset 0x2d0 contains 3 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000000020  000200000002 R_X86_64_PC32     0000000000000000 .text + 0
000000000040  000200000002 R_X86_64_PC32     0000000000000000 .text + 1a
000000000060  000200000002 R_X86_64_PC32     0000000000000000 .text + 34
No processor specific unwind information to decode

Symbol table '.symtab' contains 8 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS test.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 .text
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 .rodata
     4: 0000000000000000    26 FUNC    GLOBAL DEFAULT    1 ttt
     5: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND puts
     6: 000000000000001a    26 FUNC    GLOBAL DEFAULT    1 fff
     7: 0000000000000034    26 FUNC    GLOBAL DEFAULT    1 ggg

No version information found in this file.

Displaying notes found in: .note.gnu.property
  Owner                Data size 	Description
  GNU                  0x00000010	NT_GNU_PROPERTY_TYPE_0
      Properties: x86 feature: IBT, SHSTK
$	readelf -l test.o

There are no program headers in this file.
```

The Elf type of an object file, is `REL` in the elf header:

```
$	readelf --file-header test.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          912 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         14
  Section header string table index: 13
```

Elf Object Files can both import, and export symbols.
