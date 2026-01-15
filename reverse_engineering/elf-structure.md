# ELF Structure

#### What is an ELF File?
* Executable and linkable format
* Describes a program as it will be loaded and executed in memory.
* Stores data such as:
	* Architecture its compiled for
* Describes how the program should be loaded
* Contains metadata describing program components (section headers)
	* I think these are the assembly section headers

#### ELF Program Headers
* Program headers specify information needed to prepare the program for execution.
* Most important entry types:
	* `INTERP` - Defines the library that should be used to load this ELF into memory. 
	* `LOAD` - Defines a part of the file that should be loaded into memory. 
* Program headers are *the* source of information used when loading a file.
* All ELFs start with the bytes: `7f 45 4c 46` (spells ELF)
* Program headers describe segments that will be loaded into memory.
	* The segments may be loaded with different access rights, such as Read, Read and Execute, and Read and Write

#### What is an ELF
* File format for executable, shared libraries, and object files.
* Two terms:
	* Segments: only relevant at runtime
	* Sections: only relevant at link time
* Any elf file can contain 0 or more segments and 0 or more sections 
* It's completely valid to have an executable that only contains segments and an object file that only contains sections.
* A segment will point to some address in memory and specify how many bytes are contained in that segment.
* Sections are similar, and they may even overlap in segemtns 


#### Segments
* The difference between segments and sections is that segments also specify how to map parts of the ELF file into memory.
* There are usually two main segments for a statically linked executable:
	* Data segment: initialized globals and other initialized data.
		* Will be larger than the segment in the elf file itself, to allow for room for uninitialized data
	* Code segment: The OS will load the address of the entrypoint, the point in the executable where the binary should start running
* In a dynamically linked binary, there is an additional segment in the ELF file that specifies which shared libraries need to be loaded.
	* The OS then goes and loads those shared libraries (also ELF files and maps them into memory, just like the executable )

#### ELF Header
* The ELF header is a structure that contains the metadata of the file.
```c
typedef struct {
	unsigned char e_ident[EI_NIDENT];
	HalfWord e_type;
	HalfWord e_machine;
	Word e_Version;
	Address e_entry;
	Offset e_phoff;
	Offset e_shoff;
	Word e_flags
	HalfWord e_ehsize;
	HalfWord e_phentsize;
	HalfWord e_phnum;
	HalfWord e_shentsize; 
	HalfWord e_shnum; 
	HalfWord eshstrndx;
} ElfHeader;

HalfWord = uint16_t
Address = uint32_t/uint_64t
Word = uint32_t
```
* e_ident is 16 bytes that describes how the ELF file needs to be parsed.
	* First 4 bytes contain a magic number 0x7f followed by the characters E, L, and F encoded as ASCII
	* 5th byte describes the class of the ELF file 
		* 0 = None
		* 1 = 32 bit objects
		* 2 = 64 bit objects 
	* 6th byte describes data encoding used in the file
		* 0 = None (Invalid)
		* 1 = LSB - Least significant byte first
		* 2 = MSB - Most Significant Byte First
	* 7th byte: Version Byte
	* 8th bit: OS ABI
		* 0 = None/System V (Very common)
		* 1 = HP-UX
		* 2 = NetBDS
		* 3 = Linux
	* 9th byte: Almost ever used
	* Last 7 bytes: padding, not used at all
* e_type 
	* ET_NONE --> 0 = No file Type
	* ET_REL --> 1 = Relocatable file type (used for object files)
	* ET_EXEC --> 2 = Executable file (used for executables, does not support ASLR)
	* ET_DYN --> 3 =  shared object file (used for shared libraries and executable with position independent code)
* e_machine --> Some value that corresponds to the machine type (ex: x86, MIPS, ARM)
* e_version --> always sent to 1 
* e_entry --> Specifies entrypoint into executable or constructor address for shared libraries, if there is no entrypoint, this will be set to 0.
* e_phoff and e_shoff --> Program headers offset and section headers offset relative to beginning of file.
* e_flags --> Flags for the file, heavily architecture and OS dependent.
* e_phentsize and e_phnum --> Size per program header, number of program headers respectively.
* e_shentsize and shnum --> Size per section header, number of section headers
* e_shstrndx --> Section header string table index. 

#### Program Headers
* Each segement is described by a small program header structure.
* All program headers are in an array directly behind each other in the ELF file. 
* e_phoff member of ELF header describes the offset to the program headers.
```c
typedef struct {
	Word type;
	Offset offset;
	Address vaddr;
	Address paddr;
	Word filesz;
	Word memsz;
	Word flags;
	Word align;
} ProgramHeader32;
```
* type --> specifies the segment type.
	* 0 = PT_NULL --> placeholder and provides a simple way to disable a segment
	* 1 = PT_LOAD --> Segments of this type will be loaded into memory. How exactly it will be loaded is specified by the other members of the program header.
	* 2 = PT_DYNAMIC --> contains information required for dynamically linked binaries 
	* 3 = PT_INTERP --> specifies program interpreter required by dynamic executable. 
		* Only executables will have a shared library of this type, while shared libraries will not. 
	* 4 = PT_NOTE --> auxillary information
	* 5 = PT_SHLIB --> undefined and should never be  used
	* 6 = PT_PHDR --> Specifies whether and where program header table will be loaded into memory.
	* PT_TLS 
* offset --> Specifies in the ELF file where the content of the segment is located.
* vaddr --> specifies where the first byte of segment will be in memory
* paddr --> only used in context where physical memory is relevant
* filesz --> Size of the segment in the file. If 0 the segment is defined exclusively by the program header
* memsz --> If greater than size in file, leftover bytes will be initialized to 0 
* flags --> Defines permissions of the segment 
	* PF_X - Executable
	* PF_W - Writable
	* PF_R - Readable
* align --> Specifies whether a segment needs to be aligned to a 4 byte or an 8 byte boundary.

#### Section Headers
```c
typedef struct {
	Word sh_name; 
	Word sh_type; 
	Word sh_flags;
	Address sh_addr;
	Offset sh_offset;
	Word sh_size; 
	Word sh_link; 
	Word sh_info; 
	Word sh_addralign;
	Word sh_entsize;
} Sectionheader32;
```
* First member of section header is the name of the section, an offset to the section header string table.
	* A string table is basically an array of 0 terminated strings.
* sh_type --> Like segments can have quite some different types.
	* 3 = SHT_STRTAB
		* String table type 
* sh_flags
	* SHF_WRITE = 0x1
	* SHF_ALLOC = 0x2
	* SH_EXECINSTR = 0x3
* sh_addr --> Location of section data in ELF file
* sh_size --> how big section is
* sh_link --> allows a section to be linked to another section
* sh_info --> contains extra info depending on section typ e
* sh_addralign --> specifies whether section has any alignment constraitns
* sh_entsize --> For sections that contain fixed size entires, contains the size for each entry.

## From pwn.college video

#### What is an ELF?
* Executable and Linkable format
* defines a program as it will be loaded and executed in memory
* Contains a program and its data
* Describes how program should be loaded (program/segment headers)
* Contains metadata describing program components (section headers)

#### Program Headers
* Define segments, parts of an ELF loaded into memory
	* Most important types:
		* INTERP: defines a library that should be used to load this ELF Into memory
		* LOAD: defines a part of the file that should be loaded into memory
	* Progam headers are the source of information used when loading a file.

#### Section headers
* A different view of the ELF with useful information for introspection, debugging, etc.
* Improtant sections:
	* `.text`: the executable code of your program`
	* `.plt` and `.got`: used to resolve and dispatch library calls
		* 
	* `.data`: used for pre-initialzied global writable data (such as global arrays with initial values)
		* Global and writable data with an initial value
	* `.rodata`: used for global read-only data (such as string constants)
		* For read-only data. Thinks like string literals. 
	* `.bss`: used for uninitialized global writable data (such as global arrays without initial values)
		* 
* Section headers are not necessary part of the ELF: only segments are needed for loading and operation 
* Section headers are just metadata

#### Symbols
* Binaries (and libraries) that used dynamically loaded libraries rely on symbols (names) to find libraries, resolve function calls into those libraries, etc. 
* This is like the symbols used in assembly programming. ''

#### Interacting with your ELF
* **gcc** to make your ELF 
* **readelf** to parse the ELF header
* **objdump** to parse the ELF header and disassemble the source code.
* **nm** to view your elf's symblols
* **patchelf** to change some ELF properties 
* **objcopy** to swap out ELF sections
* **strip** to remove otherwise-helpful information (such as symbols)
* **kaitai struct** to look through your ELF interactively.
