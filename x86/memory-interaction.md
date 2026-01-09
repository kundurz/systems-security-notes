# Memory Interaction

* Memory request by CPU goes through levels of caching, then gets put into registers to be used by the CU or the ALU
* Process memory is addressed linearly 
	* From 0x1000 too 0x7ffffffff
	* Each memory address references **one byte** in memory. This means 127 terabytes of addressable RAM
* You don't have 127 TB of RAM ... But that's okay because its all virtual. 
* Your process' memory starts out partially filled by the OS.
* Your process can ask for more memory by the OS.

#### The Stack
* For temporary data storage
* Registers and immediates can be pushed onto the stack to save values. 
```asm
mov rax, 0xc001ca75
push rax
push 0xb0bacafe <<-- Even on 64-bit x86, you can only push 32-bit immediates. 
push rax
```
* Then values can be popped back off
```asm
pop rbx 
pop rcx
```
* Fun fact: this doesnt actually remove them from the stack, we just redefine where the stack ends.
* Stack grows backwards towards smaller memory addresses 

#### Accessing Memory
* You can also move data between registers and memory with `mov`
* This will load the 64-bit value stored at memory address 0x12345 into `rbx`:
```asm
mov rax, 0x12345
mov rbx, [rax] <-- will move 8 bytes, but starting from 0x12345
```
* This will store the 64-bit value in `rbx` into memory address `0x133337` 
```
mov rax, 0x13337
mov [rax], rbx
```
* This below is equivalent to `push rcx`
```asm
sub rsp, 8 <-- prepares stack, as it goes down.
mov [rsp], rcx
```
* When you use `mov` to store multi-byte values (like a 64-bit register) into memory, the bytes are stored in increasing memory addresses -- from least significant byte to most significant byte. 
* Each addressed memory location contains one byte
* An 8-byte write at address 0x133337 will write to addresses 0x133337 through 0x13333f

#### Controlling Write Sizes
* You can use partials to store/load fewer bits
* Load 64 bits form addr 0x12345 and store the lower 32 bits to addr 0x133337
```asvm
mov rax, 0x12345
mov rbx, [rax]
mov rax, 0x13337
mov [rax], ebx
```
* Load 8 bits from addr 0x12345 to `bh`
```asm
mov rax, 0x12345
mov bh, [rax]
```
* Don't forget: changing 32-bit partials (e.g., by loading from memory) zeroes out the whole 64-register. Storing 32-bits to memory has no such problems though.

#### Endianess
* Data on modern systems is stored backwards, in *little endian*
```asm
mov rax, 0xc001ca75 --> sets rax to c0 01 ca 75 
mov rcx, 0x1000
mov [rcx], eax # stores data as 75 ca 01 c0
mov bh, [rcx] # reads 0x75
```
* Bytes are only suffled for multi-byte stores and loads of registers to memory! 
* Individual bytes never have their bits shuffled
* yes, writes to the stack behave just like any other write to memory
* Why little endian
	* Intel created the 8008 for a company called Datapoint in 1972
	* Datapoint used little endian for easier implementation of carry in arithmetic
	* Intel used little endian in 8008 for compatability with Datapoint's processes
	* Every step in the evolution between 8008 and modern x86 maintained some level of binary compatability with its predecessor.
* push will write the 8 bytes in reverse

#### Address Calculation
* You can do some limited calculation for memory addresses
* Use `rax` as an offset off some base address (in case, the stack)
```
mov rax, 0 
mov rbx, [rsp+rax*8]
inc rax
mov rcx, [rsp+rax*8]
```
* You can get the calculated address with Load Effective Address (lea)
```asm
mov rax, 1
pop rcx
lea rbx, [rsp+rax*8+5] # rbx now holds the computed address for double-checking
mov rbx, [rbx]
```
* Address calculation has limits
	* reg + reg \* (2 or 4 or 8) + value 

#### RIP (as in instruction pointer) - Relative Addressing
* `lea` is one of the few instructions that can directly access the rip register
	* `lea rax, [rip]`
	* `lea rax, [rip+8]`
* You can also use `mov` to read directly from those locations
	* `mov rax, [rip]`
	* Or even write there
	* `mov [rip], rax`
* This is useful  for working with data embedded near your code
* This is what makes certain security features on modern machines possible.

#### Writing Immediate Values
* You can also write immediate values. However, you must specify their size
* This writes a 32-bit 0x1337 (padded with 0 bits) to address 0x133337
```asm
mov rax, 0x133337
mov DWORD PTR [rax], 0x1337 <-- important to specify size because then CPU will not know how many bytes to write.
```
* Depending on your assembler, it might expect DWORD instead of DWORD PTR
* Certain operations are just not supported by the architecture, like moving a 64-bit immediate value directly into memory. You have to put it in a register first, and *then* move it into memory.

#### Other Memory Regions
* Other regions might be mapped in memory.
