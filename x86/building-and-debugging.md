# Building Programs

* Assembly is assembled by an assembler into binary file.
* The first thing you have to do is specify the syntax you are using:
	* `.intel_syntax noprefix` (specifies no % before registers)
* To assemble it:
	* `gcc -nostdlib -o file file.s`
* You can define the entrypoint of your program:
```asm
.global _start
_start:
```

#### Reading Assembly
* You can disassemble your program
* `objdump -M intel -d quitter`
* gcc builds your Assembly into a full ELF program.,
* You can extract *just* your binary code.
* `objcppy --dump-section .text=file_binary_code file`

#### Debugging
* Debugging is done with debuggers, such as `gdb`
* When the `int3` instruction is executed, the debugged program is interrupted and you can inspect its state. It's just a way of setting a breakpoint in your code.
* You can also set breakpoints yourself in the debugger.
