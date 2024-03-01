## ELF Header

The ELF Header format actually changes, depending on if it is a `32` or `64` bit binary. Since `32` bit addresses are `4` bytes long, and `64` bit addresses are `8` bytes long, some of the sizes/offsets are different for when it starts getting to the offsets. The size of a `32` bit elf header is `0x34` bytes long, and for a `64` bit elf header the size is `0x40` bytes:

64 bit elf header:

| Offset | Size | Dataype | Name |
| ----- | ----- | ----- | ----- |
| 0x00 | 0x04 | Enum | e_ident[EI_MAG0] through ei_ident[EI_MAG3] |
| 0x04 | 0x01 | Enum | e_ident[EI_CLASS] |
| 0x05 | 0x01 | Enum | e_ident[EI_DATA] |
| 0x06 | 0x01 | Enum | e_ident[EI_VERSION] |
| 0x07 | 0x01 | Enum | e_ident[EI_OSABI] |
| 0x08 | 0x01 | Enum | e_ident[EI_ABIVERSION] |
| 0x09 | 0x07 | Unused | e_ident[EI_PAD] |
| 0x10 | 0x02 | Enum | e_type |
| 0x12 | 0x02 | Enum | e_machine |
| 0x14 | 0x04 | Emum | e_version |
| 0x18 | 0x08 | Elf Image File Offset | e_entry |
| 0x20 | 0x08 | Elf Image File Offset | e_phoff |
| 0x28 | 0x08 | Elf Image File Offset | e_shoff |
| 0x30 | 0x04 | Flags | e_flags |
| 0x34 | 0x02 | int (size value) | e_ehsize |
| 0x36 | 0x02 | int (size value) | e_phentsize |
| 0x38 | 0x02 | int (count value) | e_phnum |
| 0x3a | 0x02 | int (size value) | e_shentsize |
| 0x3c | 0x02 | int (count value) | e_shnum |
| 0x3e | 0x02 | int (index value) | e_shstrndx |

32 bit elf header:

| Offset | Size | Dataype | Name |
| ----- | ----- | ----- | ----- |
| 0x00 | 0x04 | Enum | e_ident[EI_MAG0] through ei_ident[EI_MAG3] |
| 0x04 | 0x01 | Enum | e_ident[EI_CLASS] |
| 0x05 | 0x01 | Enum | e_ident[EI_DATA] |
| 0x06 | 0x01 | Enum | e_ident[EI_VERSION] |
| 0x07 | 0x01 | Enum | e_ident[EI_OSABI] |
| 0x08 | 0x01 | Enum | e_ident[EI_ABIVERSION] |
| 0x09 | 0x07 | Unused | e_ident[EI_PAD] |
| 0x10 | 0x02 | Enum | e_type |
| 0x12 | 0x02 | Enum | e_machine |
| 0x14 | 0x04 | Emum | e_version |
| 0x18 | 0x04 | Elf Image File Offset | e_entry |
| 0x1c | 0x04 | Elf Image File Offset | e_phoff |
| 0x20 | 0x04 | Elf Image File Offset | e_shoff |
| 0x24 | 0x04 | Flags | e_flags |
| 0x28 | 0x02 | int (size value) | e_ehsize |
| 0x2a | 0x02 | int (size value) | e_phentsize |
| 0x2c | 0x02 | int (count value) | e_phnum |
| 0x2e | 0x02 | int (size value) | e_shentsize |
| 0x30 | 0x02 | int (count value) | e_shnum |
| 0x32 | 0x02 | int (index value) | e_shstrndx |

##### e_ident[EI_MAG0] through ei_ident[EI_MAG3]

This (and everything else with `e_ident[<field>]`) are members of the `e_ident` datastructure. The `e_ident` datastructure is used to model particular things to identify the elf.

The `EI_MAG` values are just the `4` magic bytes, placed at the start of the binary image file, and used to identify that it is an elf file. These `4` bytes are:

```
7f 45 4c 46
```

It's just `0x7f` followed by `ELF` in ascii. These four bytes should always be there at the start of the file, and be that value (a constant value).

##### e_ident[E_CLASS]

This is a `1` byte value, used to specify if the ELF binary is a `32` or `64` bit binary. It is a `1` for `32` bit, and a `2` for `64` bit.

##### e_ident[E_DATA]

This is a `1` byte value, used to specify the endianess of a binary. If `1` it means it is little endian, and `2` means big endian. This byte order starts to apply to multibyte values, beginning at offset `0x10` from the start of the ELF Image FIle.

##### e_ident[E_VERSION]

This value should be set to `1`. It means, that the version of ELF this is, is the original type. There is currently only one valid value for this, as there hasn't been a new version of an elf.

##### e_ident[EI_OSABI]

An operating system's ABI (Application Binary Interface), specifies how the actual assembly code should interface with the hardware of the system. This field specifies which OS's ABI it should use:

| ABI NAME | Value |
| ----- | ----- |
| System V (most common it looks like) | 0x00 |
| HP-UX | 0x01 |
| NetBSD | 0x02 |
| Linux | 0x03 |
| GNU Hurd | 0x04 |
| Solaris | 0x06 |
| AIX (Monterey) | 0x07 |
| IRIX | 0x08 |
| FreeBSD | 0x09 |
| Tru64 | 0x0A |
| Novel Modesto | 0x0B |
| OpenBSD | 0x0C |
| OpenVMS | 0x0D |
| NonStop Kernel | 0x0E |
| AROS | 0x0F |
| FentixOS | 0x10 |
| Nuxi CloudABI | 0x11 |
| Statis Technologies OpenVOS | 0x12 |

##### e_ident[EI_ABIVERSION]

So this field, will speicify which version of the ABI is being used. So `EI_OSABI` will specify which ABI is being used, and `EI_ABIVERSION` will specify which specific version of the ABI is being used. Depending on which ABI is being used and the conditions around it, additional meaning may be tied to this value.

##### e_ident[EI_PAD]

This is just padding bytes, which are currently unuded. These should all be null (`0`).

##### e_type

This is a 2 byte value, which designates what type of elf file it is. Here is a chart:

| Value | Type | Meaning |
| ----- | ----- | ----- |
| 0x0000 | ET_NONE | File type is unknown |
| 0x0001 | ET_REL | This is an object file (`.o`) |
| 0x0002 | ET_EXEC | An executable ELF File |
| 0x0003 | ET_DYN | This is a shared object file (`.so`), and can also be an executable with PIE |
| 0x0004 | ET_CORE | A Core Dump File |
| 0xFE00 | ET_LOOS | This is a part of a range of values, ranging from ET_LOOS to ET_HIOS, that are for OS specific ELF Filetypes. |
| 0xFEFF | ET_HIOS | See above |
| 0xFF00 | ET_LOPROC | This is a part of a range of values, ranging from ET_LOPROC to ET_HIPROC, that are for Processor specific ELF Filetypes. |
| 0xFFFF | ET_HIPROC | See above |
 

##### e_machine

This value specifies the type of machine that the ELF is supposed to run on. Here is a chart, but it is basically copied from: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format).

| Value | Machine |
| ----- | ----- |
| 0x01 | AT&T WE 32100 |
| 0x02 | SPARC |
| 0x03 | x86 |
| 0x04 | Motorola 68000 (M68k) |
| 0x05 | Motorola 88000 (M88k) |
| 0x06 | Intel MCU |
| 0x07 | Intel 80860 |
| 0x08 | MIPS |
| 0x09 | IBM System/370 |
| 0x0A | MIPS RS3000 Little-endian |
| 0x0B through 0x0E | Reserved for future use |
| 0x0F | Hewlett-Packard PA-RISC |
| 0x13 | Intel 80960 |
| 0x14 | PowerPC |
| 0x15 | PowerPC (64-bit) |
| 0x16 | S390, including S390x |
| 0x17 | IBM SPU/SPC |
| 0x18 through 0x23 | Reserved for future use |
| 0x24 | NEC V800 |
| 0x25 | Fujitsu FR20 |
| 0x26 | TRW RH-32 |
| 0x27 | Motorola RCE |
| 0x28 | Arm (up to Armv7/AArch32) |
| 0x29 | Digital Alpha |
| 0x2A | SuperH |
| 0x2B | SPARC Version 9 |
| 0x2C | Siemens TriCore embedded processor |
| 0x2D | Argonaut RISC Core |
| 0x2E | Hitachi H8/300 |
| 0x2F | Hitachi H8/300H |
| 0x30 | Hitachi H8S |
| 0x31 | Hitachi H8/500 |
| 0x32 | IA-64 |
| 0x33 | Stanford MIPS-X |
| 0x34 | Motorola ColdFire |
| 0x35 | Motorola M68HC12 |
| 0x36 | Fujitsu MMA Multimedia Accelerator |
| 0x37 | Siemens PCP |
| 0x38 | Sony nCPU embedded RISC processor |
| 0x39 | Denso NDR1 microprocessor |
| 0x3A | Motorola Star*Core processor |
| 0x3B | Toyota ME16 processor |
| 0x3C | STMicroelectronics ST100 processor |
| 0x3D | Advanced Logic Corp. TinyJ embedded processor family |
| 0x3E | AMD x86-64 |
| 0x3F | Sony DSP Processor |
| 0x40 | Digital Equipment Corp. PDP-10 |
| 0x41 | Digital Equipment Corp. PDP-11 |
| 0x42 | Siemens FX66 microcontroller |
| 0x43 | STMicroelectronics ST9+ 8/16 bit microcontroller |
| 0x44 | STMicroelectronics ST7 8-bit microcontroller |
| 0x45 | Motorola MC68HC16 Microcontroller |
| 0x46 | Motorola MC68HC11 Microcontroller |
| 0x47 | Motorola MC68HC08 Microcontroller |
| 0x48 | Motorola MC68HC05 Microcontroller |
| 0x49 | Silicon Graphics SVx |
| 0x4A | STMicroelectronics ST19 8-bit microcontroller |
| 0x4B | Digital VAX |
| 0x4C | Axis Communications 32-bit embedded processor |
| 0x4D | Infineon Technologies 32-bit embedded processor |
| 0x4E | Element 14 64-bit DSP Processor |
| 0x4F | LSI Logic 16-bit DSP Processor |
| 0x8C | TMS320C6000 Family |
| 0xAF | MCST Elbrus e2k |
| 0xB7 | Arm 64-bits (Armv8/AArch64) |
| 0xDC | Zilog Z80 |
| 0xF3 | RISC-V |
| 0xF7 | Berkeley Packet Filter |
| 0x101 | WDC 65C816 |

##### e_version

This specifies the version of ELF which this is. There is currently only one version, so this value should be `1`. This is basically a copy of e_ident[EI_VERSION]

##### e_entry

This is the memory address of the first instruction address which will be executed.

##### e_phoff

This is the offset from the start of the Elf Image File to the Program Header Table.

##### e_shoff

This is the offset from the start of the Elf Image File to the Section Header Table.

##### e_flags

This contains processor-specific flags associated with the ELF file. As of right now, no flags have been defined, so this is normally just `0x00`.

##### e_ehsize

This is the size of the Elf Header in bytes, although it should either be `0x34` (`32` bit) or `0x40` (`64` bit).

##### e_phentsize

This contains the size, in bytes, of an entry of the program header table. This should be either `0x38` (`64` bit) or `0x20` (`32` bit).

##### e_phnum

This is the number of entries in the Program Header Table. The total size of the Program Header Table can be calculated via `e_phnum * e_phentsize`.

##### e_shentsize

This contains the size, in bytes, of an entry of the section header table. This should be either `0x40` (`64` bit) or `0x28` (`32` bit).

##### e_shnum

This is the number of entries in the Section Header Table. The total size of the Section Header Table can be calculated via `e_shnum * e_shentsize`.

##### e_shstrndx

This contains the index, of the section header table entry, that contains the names of the sections (will be referenced later).
