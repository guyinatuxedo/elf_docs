## What is Linking / Loading?

So the ELF File format is based around these concepts of linking and loading. Linking is the process of combining different ELF Files together, into a singular image. Loading is the process of actually loading an ELF Image File into memory for executing.

Linking can happen either at compile time (static linking), or runtime (dynamic linking).

Now the process of linking/loading is typically done by linkers and loaders, however in modern linux system these jobs got both combined in `ld.so`, which we see here (`/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2`):

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
0x00007ffff7fa6000 0x00007ffff7fbb000 0x0000000000000000 r-- /etc/ld.so.cache
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
```

The process of linking/loading depends on what linkers/loaders are used, which depends on the OS.
