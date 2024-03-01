## Program Header Table

So the program header table (PHT), is a table of individual entries. Each entry describes a segment. The table is present in the Elf Image File, at the offset specified by `e_phoff`. The size of an entry to the table is specified by `e_phentsize` (although in practice, you should know the size of an entry based on if the binary is a `32/64` bit), and the number of entries in the table is specified by `e_phnum`.

Similar to the elf header, the size of a PHT entry varies depending if it is a `32` or `64` bit binary, due to the difference in the size of addresses.

64 bit PHT entry:

| Offset | Size | Dataype | Name |
| ----- | ----- | ----- | ----- |
| 0x00 | 0x04 | Enum | p_type |
| 0x04 | 0x04 | Enum | p_flags |
| 0x08 | 0x08 | Elf Image File Offset | p_offset |
| 0x10 | 0x08 | Virtual Address | p_vaddr |
| 0x18 | 0x08 | Physical Address | p_paddr |
| 0x20 | 0x08 | Int (size) | p_filesz |
| 0x28 | 0x08 | Int (size) | p_memsz |
| 0x30 | 0x08 | Int | p_align |

32 bit PHT entry:

| Offset | Size | Dataype | Name |
| ----- | ----- | ----- | ----- |
| 0x00 | 0x04 | Enum | p_type |
| 0x04 | 0x04 | Elf Image File Offset | p_offset |
| 0x08 | 0x04 | Virtual Address | p_vaddr |
| 0x0c | 0x04 | Physical Address | p_paddr |
| 0x10 | 0x08 | Int (size) | p_filesz |
| 0x14 | 0x04 | Int (size) | p_memsz |
| 0x18 | 0x04 | Enum | p_flags |
| 0x1c | 0x04 | Int | p_align |

##### p_type

This specifies the type of segment.

| Value | Name | Meaning |
| ----- | ----- | ----- |
| 0x00000000 | PT_NULL | This Segment is unused, and this entry should be ignored. |
| 0x00000001 | PT_LOAD | This specifies a segment, that can be loaded into memory. |
| 0x00000002 | PT_DYNAMIC | This segment is present, when this object file participates in dynamic linking. This segment will contain the `.dynamic` section, for information regarding dynamic linking. |
| 0x00000003 | PT_INTERP | This segment will specify the path to the program interpreter. Executable files must have this semgent, and it can only appear once. |
| 0x00000004 | PT_NOTE | This segment type will contain speicific information, which can be used by software that deals with ELFS, to check for things like versioning / other identifieng information. |
| 0x00000005 | PT_SHLIB | This segment type isn't actually defined, and is used for special cases that do not conform to this standard. |
| 0x00000006 | PT_PHDR | This semgent type will contain the Program Header Table, and specify it's size and location. |
| 0x00000007 | PT_TLS | This segment is used to define individual copies of data (defined at compile time) to specific threads, with both size and contents. |
| 0x60000000 | PT_LOOS | So, there is a range, for Operating System (OS) specific `p_types`. This is the range between `PT_LOOS` to `PT_HIOS`. |
| 0x6fffffff | PT_HIOS | See above |
| 0x60000000 | PT_LOPROC | So, there is a range, for Processor (proc) specific `p_types`. This is the range between `PT_LOPROC` to `PT_HIPROC`. |
| 0x6fffffff | PT_HIPROC | See above |


##### p_flags

This value, consists of flags. These flags primarily describe the memory permissions, of the segment after it gets loaded into memory:

| Bit Flag | Name | Meaning |
| ----- | ----- | ----- |
| 0x01 | PF_X | This memory is executable |
| 0x02 | PF_W | This memory is writeable |
| 0x04 | PF_R | This memory is readable |

##### p_offset

This is the offset, from the start of the Elf Image File, to the start of the data for the segment.

##### p_vaddr

This is the virtual address of the start of the segment, when it gets loaded into memory.

##### p_paddr

If the system the elf is used on has physical addresses which are relevant to the elfs, this field specifies the start of the segment when loaded into memory.

##### p_filesz

For the segment data from the Elf Image File, this specifies the size of that data.

##### p_memsz

This specifies the size of the segment in memory.

##### p_align

This specifies the alignment of the segment. If this value is either `0` or `1`, it means there is no alignment. If this value is set, then `(p_vaddr % p_align) = (p_offset % p_align)` with `%` being the modulo operator. It must be a power of 2, if the alignment is set.
