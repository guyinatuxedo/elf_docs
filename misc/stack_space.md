# Where does the Stack Space Come From?

So, where does the space for the stack come from. By that I mean the memory for the stack region of memory:

```
gef➤  vmmap
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x0000555555554000 0x0000555555555000 0x0000000000000000 r-- /elf/try
0x0000555555555000 0x0000555555556000 0x0000000000001000 r-x /elf/try
0x0000555555556000 0x0000555555557000 0x0000000000002000 r-- /elf/try
0x0000555555557000 0x0000555555558000 0x0000000000002000 r-- /elf/try
0x0000555555558000 0x0000555555559000 0x0000000000003000 rw- /elf/try
0x00007ffff7c00000 0x00007ffff7c28000 0x0000000000000000 r-- /usr/lib/x86_64-linux-gnu/libc.so.6
0x00007ffff7c28000 0x00007ffff7dbd000 0x0000000000028000 r-x /usr/lib/x86_64-linux-gnu/libc.so.6
0x00007ffff7dbd000 0x00007ffff7e15000 0x00000000001bd000 r-- /usr/lib/x86_64-linux-gnu/libc.so.6
0x00007ffff7e15000 0x00007ffff7e16000 0x0000000000215000 --- /usr/lib/x86_64-linux-gnu/libc.so.6
0x00007ffff7e16000 0x00007ffff7e1a000 0x0000000000215000 r-- /usr/lib/x86_64-linux-gnu/libc.so.6
0x00007ffff7e1a000 0x00007ffff7e1c000 0x0000000000219000 rw- /usr/lib/x86_64-linux-gnu/libc.so.6
0x00007ffff7e1c000 0x00007ffff7e29000 0x0000000000000000 rw- 
0x00007ffff7fa3000 0x00007ffff7fa6000 0x0000000000000000 rw- 
0x00007ffff7fbb000 0x00007ffff7fbd000 0x0000000000000000 rw- 
0x00007ffff7fbd000 0x00007ffff7fc1000 0x0000000000000000 r-- [vvar]
0x00007ffff7fc1000 0x00007ffff7fc3000 0x0000000000000000 r-x [vdso]
0x00007ffff7fc3000 0x00007ffff7fc5000 0x0000000000000000 r-- /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x00007ffff7fc5000 0x00007ffff7fef000 0x0000000000002000 r-x /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x00007ffff7fef000 0x00007ffff7ffa000 0x000000000002c000 r-- /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x00007ffff7ffb000 0x00007ffff7ffd000 0x0000000000037000 r-- /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x00007ffff7ffd000 0x00007ffff7fff000 0x0000000000039000 rw- /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
0x00007ffffffde000 0x00007ffffffff000 0x0000000000000000 rw- [stack]
0xffffffffff600000 0xffffffffff601000 0x0000000000000000 --x [vsyscall]
gef➤  i r
rax            0x555555555169      0x555555555169
rbx            0x0                 0x0
rcx            0x555555557db8      0x555555557db8
rdx            0x7fffffffe128      0x7fffffffe128
rsi            0x7fffffffe118      0x7fffffffe118
rdi            0x1                 0x1
rbp            0x1                 0x1
rsp            0x7fffffffe008      0x7fffffffe008
r8             0x7ffff7e1bf10      0x7ffff7e1bf10
r9             0x7ffff7fc9040      0x7ffff7fc9040
r10            0x7ffff7fc3908      0x7ffff7fc3908
r11            0x7ffff7fde660      0x7ffff7fde660
r12            0x7fffffffe118      0x7fffffffe118
r13            0x555555555169      0x555555555169
r14            0x555555557db8      0x555555557db8
r15            0x7ffff7ffd040      0x7ffff7ffd040
rip            0x555555555169      0x555555555169 <main>
eflags         0x246               [ PF ZF IF ]
cs             0x33                0x33
ss             0x2b                0x2b
ds             0x0                 0x0
es             0x0                 0x0
fs             0x0                 0x0
gs             0x0                 0x0
```

What actually specifies that the stack is going to be between `0x00007ffffffde000` and `0x00007ffffffff000` (`0x21000` bytes in total)?

After a bit of googling, the answer I'm arriving to is it isn't determined by the ELF format, rather the OS. The stack starts at `0x00007ffffffff000` and grows downwards. This is evident from the fact that in function prologues it decrements the stack ptr:

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
```

Basically, while determined by the OS, it will initialize a base amount of memory for the stack. When the stack grows too big, it will trigger special types of exceptions, which will be the OS's signal to allocate more space to the stack.
