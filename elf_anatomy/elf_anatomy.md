# Anatomy of an Elf File

ELF (Executable and Linkable Format) is the file format used by executables in linux.

So the file which stores an ELF, is known as an Elf Image File. An Elf Image File, will contain three seperate headers. The first is the generic elf header, which is at the start of the Elf Image File. Elfs have these things called segments and sections. The next two headers each, are used to either describe all of the segments, or all of the sections in an elf. These two headers are both tables, with each entry into the table modeling either a section or a segment (depending on which table it is in).

## Segments and Sections

Segments and Sections, both describe pieces of the elf binary. The difference is, segments describe pieces that are used at runtime (when the binary is actually being ran). Sections contain pieces that are used at link time (when the binary is being linked together with something else either being compiled, or ran). There can be overlap between segments and sections. Segments can contain `0` or more sections.

The two other headers I describe earlier, are the Program Header Table (PHT) and the Section Header Table (SHT). The PHT describes the various segments in the elf, whereas the SHT describes the various sections in the elf.



