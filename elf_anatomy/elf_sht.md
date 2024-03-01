## Section Header Table

So the SHT (Section Header Table) is a table of entries, where each entry specifies a different section that the binary has. The table is present in the ELF Image File, at the offset specified by `e_shoff`. The number of entries in the table is specified by `e_shnum`, with the size of an entry being specified by `e_shentsize` (although in practice, you should know the size of an entry based on if the binary is a `32/64` bit).

Similar to the elf header, the size of a SHT entry varies depending if it is a `32` or `64` bit binary, due to the difference in the size of addresses.

64 bit SHT entry:

| Offset | Size | Dataype | Name |
| ----- | ----- | ----- | ----- |
| 0x00 | 0x04 | Int (offset) | sh_name |
| 0x04 | 0x04 | Enum | sh_type |
| 0x08 | 0x08 | Flags | sh_flags |
| 0x10 | 0x08 | Virtual Address | sh_addr |
| 0x18 | 0x08 | Elf Image File Offset | sh_offset |
| 0x20 | 0x08 | Int (size) | sh_size |
| 0x28 | 0x04 | Int (index) | sh_link |
| 0x2c | 0x04 | Info | sh_info |
| 0x30 | 0x08 | Int | sh_addralign |
| 0x38 | 0x08 | Int (size) | sh_entsize |

32 bit SHT entry:

| Offset | Size | Dataype | Name |
| ----- | ----- | ----- | ----- |
| 0x00 | 0x04 | Int (offset) | sh_name |
| 0x04 | 0x04 | Enum | sh_type |
| 0x08 | 0x04 | Flags | sh_flags |
| 0x0c | 0x04 | Virtual Address | sh_addr |
| 0x10 | 0x04 | Elf Image File Offset | sh_offset |
| 0x14 | 0x04 | Int (size) | sh_size |
| 0x18 | 0x04 | Int (index) | sh_link |
| 0x1c | 0x04 | Info | sh_info |
| 0x20 | 0x04 | Int | sh_addralign |
| 0x24 | 0x04 | Int (size) | sh_entsize |

##### sh_name

This is a `0x04` byte value. It serves as an offset from the start of the section (`.shstrtab`) which contains the names of the various sections. The offset is to a null terminated string, which is the name of the section.

The Elf Header has a `e_shstrndx` field, which specifies the index of the section in the Section Header Table which contains the section name.

##### sh_type

This is a `0x04` byte enum value, which designates what type of section this is. Here is the chart:

| Value | Name | Meaning |
| ----- | ----- | ----- |
| 0x00 | SHT_NULL | Section Header Table Entry is unused |
| 0x01 | SHT_PROGBITS | Contains data relevant to the program itself (includes runnable code, global data, got, etc.) |
| 0x02 | SHT_SYMTAB | This section contains a symbol table, which is used for linking. |
| 0x03 | SHT_STRTAB | This section contains a table of strings, there can be multiple of these in the same object file. |
| 0x04 | SHT_RELA | This section contains relocation data (excludes plt relocations). |
| 0x05 | SHT_HASH | This section is a hashtable for symbols. |
| 0x06 | SHT_DYNAMIC | This section is a table, which has entries that specify dynamic linking information. |
| 0x07 | SHT_NOTE | This section contains miscellaneous information, that isn't actually needed for elf loading/linking/running. |
| 0x08 | SHT_NOBITS | This section does not occupy any space in the ELF Image File (used for stuff like `.bss`, which is initialized to `0x00`) |
| 0x09 | SHT_REL | This section is a table, which has entries that specify relocation information. |
| 0x0a | SHT_SHLIB | This section is reserved for future use. |
| 0x0b | SHT_DYNSYM | This holds symbols to global things (like functions) which will be dynamically linked. |
| 0x0e | SHT_INIT_ARRAY | This section contains an array of pointers, to functions, which will be executed on program initialization. |
| 0x0f | SHT_FINI_ARRAY | This section contains an array of pointers, to functions, which will be executed when the program finishes. |
| 0x10 | SHT_PREINIT_ARRAY | This section contains an array of pointers to functions, which will be executed before `SHT_INIT_ARRAY` functions. |
| 0x11 | SHT_GROUP | Specifies a group of sections that must remained group together, if a link editor changes how linking works with this ELF. |
| 0x12 | SHT_SYMTAB_SHNDX | So in some instances, the index into `SHT_SYMTAB` might be too large, in which case it would be stored in this section. |
| 0x13 | SHT_NUM | This section can contain a number of defined types |

There are other section types not mentioned here, such as `SHT_LOOS` (`0x60000000`).

##### sh_flags

This field contains flags, which apply to the section. Here are some of the flags:

| Flag Bit | Flag Name | Meaning |
| ----- | ----- | ----- |
| 0x01 | SHF_WRITE | This section's data should be writeable durring program execution |
| 0x02 | SHF_ALLOC | This segment will occupy memory durring program execution |
| 0x04 | SHF_EXECINSTR | This segment contains executable instructions |
| 0x10 | SHF_MERGE | This segment can be merged with others |
| 0x20 | SHF_STRINGS | This segment contains null-terminated strings |
| 0x40 | SHF_INFO_LINK | The `sh_info` field contains the SHT index|
| 0x80 | SHF_LINK_ORDER | This flag is used sometimes when this section references others, that they must appear in the output in the same relative order (will be discussed later) |
| 0x100 | SHF_OS_NONCOMFORMING | This section requires non-standard OS Specific Handling |
| 0x200 | SHF_GROUP | This section is a member of a group (will be discussed later)  |
| 0x400 | SHF_TLS | This section will be used for thread local storage |
| 0x0ff00000 | SHF_MASKOS | OS specific flags |
| 0xf0000000 | SHF_MASKPROC | Processor Specific flags |
| 0x4000000 | SHF_ORDERED | Special ordering is required with this linking this section (primarily used with Solaris) |
| 0x8000000 | SHF_EXCLUDE | Unless this section is referenced or allocated, exclude it (primarily used with Solaris) |

##### sh_addr

This is the virtual address, which this section should start at.

##### sh_offset

For the section data that is present in the ELF Image File, this specifies the offset from the start of the ELF Image File to that data.

##### sh_size

This specifies the size, in bytes, of the section data present in the ELF Image File.

##### sh_link

If there is another section which is associated with this section, this field will contain the index of that section (for the Section Header Table). Depending on what type of section this is, it can mean different things.

##### sh_info

This field will contain various different pieces of information, depending on what type of section we are dealing with.

##### sh_addralign

This field contains the required address alignment for this section. It must be a power of 2.

##### sh_entsize

If this section will contain fixed sized pieces of data, this field contains the size of one of those pieces. Otherwise, this value should be `0x00`.
