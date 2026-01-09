# Control Flow + Calling Conventions

#### What to Execute?
* Different number of bytes to encode each instruction. This is a variable-length instruction set architecture (ISA)

#### Jumps
* CPUs execute instructions in sequence *until told not to*
* One way to interrupt the sequence is with the `jmp` instruction
```asm
	mov cx, 1337
	jmp STAY_LEET
	mov cx, 0 <-- Never executes 
STAY_LEET:
	push rcx
```
* jmp skips X bytes and then resumes execution!

#### Control Flow: Conditions
* Conditional jumps check conditions stored in the "flags" register: `rflags`
* Comparison instructions update the flags.
	* `cmp` (sub, but discards result)
	* `test` (`and`, but discards result)
* Main conditional flags:
	* Carry flag: Was the 65th bit 1?
	* Zero flag: Was the result 0?
	* Overflow flag: did the result "wrap" between positive to negative
	* Signed flag: Was the result's sign bit set (i.e., was it negative)?
	* Rarely do you interact with these flags directly, you usually use some semantic stuff like `je` or `jne` or `jg` (conditional jumps)
* Common patterns:
```asm
cmp rax, rbx; ja STAY_LEET # unsigned rax > rbx. 0xffffffff (jump if above)
cmp rax, rbx; jle STAY_LEET # signed rax <= rbx 
test rax, rax; jnz STAY_LEET # rax != 0
cmp rax, rbx; je STAY_LEET # rax == rbx
```

* Thanks to Two's Complement, only the jumps themselves have to be signedness-aware
#### Conditional Jumps
* Jumps can rely on conditions
```asm
mov cx, 1337
jnz STAY_LEET
mov cx, 0
STAY_LEET:
push rcx
```

#### Looping
* With our conditional jumps, we can implement a loop
* Example: this counts to 10!
```asm
mov rax, 0
LOOP_HEADER:
inc rax
cmp rax, 10
jb LOOP_HEADER
```

#### Control Flow: Function Calls!
* Assembly code is split into functions with `call` and `ret`
* `call` pushes `rip` (address of the next instruction after the call) and jumps away.
* `ret` pops `rip` and jumps to it

#### Calling Conventions
* Linux x86: push arguments (in reverse order), then call (which  pushes reutrn address), return value eax
* Linux amd64: rdi, rsi, rdx, rcx, r8, r9, return value in rax
* Linux arm: r0, r1, r2, r3, return value in r0
* Registers are shared between functions, so calling conventions should agree on which registers are protected
