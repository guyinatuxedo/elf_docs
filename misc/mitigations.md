# Binary Mitigations

So, in some of my earlier docs, I discussed how some of the various binary mitigations work. This is just a document to briefly describe how some of the common binary mitigations work in one spot, which will rehash some of that information.

## Stack Canary

The Stack Canary is a binary mitigation, designed to make using a stack buffer overflow bug harder to leverage. It does this via saving a piece of data to the stack when the function starts, and then before the function ends it checks the stack canary to see if it changed.

So, the stack canary is implemented at compile time. For every function deemed in need of stack canary protection, the compiler will implement the instructions at the start of the function and before the function executes the saved stack return address to check for a stack smash.

Here we see an example of a stack canary check being implemented:

```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined main()
             undefined         AL:1           <RETURN>
             undefined8        Stack[-0x10]:8 local_10                                XREF[2]:     0010117e(W), 
                                                                                                   0010119d(R)  
             undefined1        Stack[-0x78]:1 local_78                                XREF[1]:     0010118b(*)  
                             main                                            XREF[4]:     Entry Point(*), 
                                                                                          _start:00101098(*), 00102030, 
                                                                                          001020c8(*)  
        00101169 f3 0f 1e fa     ENDBR64
        0010116d 55              PUSH       RBP
        0010116e 48 89 e5        MOV        RBP,RSP
        00101171 48 83 ec 70     SUB        RSP,0x70
        00101175 64 48 8b        MOV        RAX,qword ptr FS:[0x28]
                 04 25 28 
                 00 00 00
        0010117e 48 89 45 f8     MOV        qword ptr [RBP + local_10],RAX
        00101182 31 c0           XOR        EAX,EAX
        00101184 48 8b 15        MOV        RDX,qword ptr [stdin]
                 85 2e 00 00
        0010118b 48 8d 45 90     LEA        RAX=>local_78,[RBP + -0x70]
        0010118f be 63 00        MOV        ESI,0x63
                 00 00
        00101194 48 89 c7        MOV        RDI,RAX
        00101197 e8 d4 fe        CALL       libc.so.6::fgets                                 char * fgets(char * __s, int __n
                 ff ff
        0010119c 90              NOP
        0010119d 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        001011a1 64 48 2b        SUB        RAX,qword ptr FS:[0x28]
                 04 25 28 
                 00 00 00
        001011aa 74 05           JZ         LAB_001011b1
        001011ac e8 af fe        CALL       libc.so.6::__stack_chk_fail                      undefined __stack_chk_fail()
                 ff ff
                             -- Flow Override: CALL_RETURN (CALL_TERMINATOR)
                             LAB_001011b1                                    XREF[1]:     001011aa(j)  
        001011b1 c9              LEAVE
        001011b2 c3              RET
```

Note the `MOV` instructions at `0x00101175/0x0010117e`. This is it moving the stack canary to the stack. Then at `0x001011a1/0x001011aa` is the comparison to see if the stack canary has been changed. If it has, the `libc.so.6::__stack_chk_fail` function is called to report the stack canary change (and kill the program).

Functions that do not have stack buffers that are written to can be deemed as uneeding of a Stack Canary check (since adding this check in results in a performance hit). We see here, this binary has the Stack Canary enabled:

```
$	pwn checksec try
[*] '/elf/try'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

However we see the `main` function does not have the stack canary protection:

```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined main()
             undefined         AL:1           <RETURN>
             undefined1        Stack[-0xd]:1  local_d                                 XREF[1]:     00101175(*)  
                             main                                            XREF[4]:     Entry Point(*), 
                                                                                          _start:00101098(*), 00102038, 
                                                                                          001020d0(*)  
        00101169 f3 0f 1e fa     ENDBR64
        0010116d 55              PUSH       RBP
        0010116e 48 89 e5        MOV        RBP,RSP
        00101171 48 83 ec 10     SUB        RSP,0x10
        00101175 48 8d 45 fb     LEA        RAX=>local_d,[RBP + -0x5]
        00101179 48 89 c7        MOV        RDI,RAX
        0010117c b8 00 00        MOV        EAX,0x0
                 00 00
        00101181 e8 ea fe        CALL       <EXTERNAL>::gets                                 char * gets(char * __s)
                 ff ff
        00101186 48 8d 05        LEA        RAX,[DAT_00102004]                               = 54h    T
                 77 0e 00 00
        0010118d 48 89 c7        MOV        RDI=>DAT_00102004,RAX                            = 54h    T
        00101190 e8 cb fe        CALL       <EXTERNAL>::puts                                 int puts(char * __s)
                 ff ff
        00101195 90              NOP
        00101196 c9              LEAVE
        00101197 c3              RET
```

## Stack NX

So a Non-Executable Stack basically means the memory permissions on the stack do not include the Executable Bit (typically just read and write). This means that you cannot execute data in the stack memory region as instructions.

This is specified with the `PT_GNU_STACK` segment documented earlier, which it's `p_flags` will specify it's memory permissions (typically `6` for read/write, standard linux file permissions where `4` means read, `2` means write, and `1` means executable). If this semgent is not included, than the Stack will be made executable.

What actually assigns the memory permissions to the stack, is the elf loader. As it parses the segments, it will see this segment, and assign memory permissions to the stack as a result.

Here is an example of a `PT_GNU_STACK` semgent:

```
           001002a8 51 e5 74 64 06  Elf64_Phdr                        [11]
                    00 00 00 00 00 
                    00 00 00 00 00
              001002a8 51 e5 74 64     Elf_Prog  PT_GNU_STACK            p_type        PT_GNU_STACK - Ind
              001002ac 06 00 00 00     ddw       6h                      p_flags
              001002b0 00 00 00 00 00  dq        0h                      p_offset
                       00 00 00
              001002b8 00 00 00 00 00  dq        0h                      p_vaddr
                       00 00 00
              001002c0 00 00 00 00 00  dq        0h                      p_paddr
                       00 00 00
              001002c8 00 00 00 00 00  dq        0h                      p_filesz
                       00 00 00
              001002d0 00 00 00 00 00  dq        0h                      p_memsz
                       00 00 00
              001002d8 10 00 00 00 00  dq        10h                     p_align
                       00 00 00
```

## Relro

RELRO (Read only Relocations) is a binary mitigation. In binaries, espescially when dealing with linked data to shared libraries, the ELF won't actually know the address of the data until runtime.

That is where relocations come into play. Relocations are effectively what resolves the address in these instances. There are different types of relocations, where it can occur only when the address is needed, or all at once.

One popular type of exploit, is to target these addresses, since they are writeable instruction pointers. That is where RELRO comes into play. With this, all the relocations happen when the binary is loading/dynamically linking, and afterwards is marked as read only so they can't be overwritten.

This is specified with the `PT_GNU_RELRO` segment, which was documented earlier. This segment will describe the starting address and size of the relocation area to make read only after relocations happen (since in order for relocations to happen, the area must be writeable):

```
           001002e0 52 e5 74 64 04  Elf64_Phdr                        [12]
                    00 00 00 b0 2d 
                    00 00 00 00 00
              001002e0 52 e5 74 64     Elf_Prog  PT_GNU_RELRO            p_type        PT_GNU_RELRO - Spe
              001002e4 04 00 00 00     ddw       4h                      p_flags
              001002e8 b0 2d 00 00 00  dq        2DB0h                   p_offset
                       00 00 00
              001002f0 b0 3d 00 00 00  dq        __frame_dummy_init_arr  p_vaddr       = 101160h
                       00 00 00
              001002f8 b0 3d 00 00 00  dq        3DB0h                   p_paddr
                       00 00 00
              00100300 50 02 00 00 00  dq        250h                    p_filesz
                       00 00 00
              00100308 50 02 00 00 00  dq        250h                    p_memsz
                       00 00 00
              00100310 01 00 00 00 00  dq        1h                      p_align
                       00 00 00
```

## Stack/Heap/Shared Library ASLR

So ASLR is a binary mitigation, designed to change the start of various memory regions every time the binary runs.

This is implemented in the kernel, which is repsonsible for managing virtual memory. It is typically controlled by the value stored in the `/proc/sys/kernel/randomize_va_space` file in linux.

## PIE

PIE (Position Independent Executable) is ASLR, for the actual "binary" data (where the instructions / global variables / etc) goes.

In order to implement PIE, the elf type is changed from `ET_EXEC` to `ET_DYN` in the elf header. This will effectively change the elf from an executable, to a shared library, and it will receive ASLR similar to a shared library. The randmoized base (which all addresses in the elf are offsets from the base) is determined by the loader.
