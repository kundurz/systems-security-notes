# functions and frames

#### What is a program?
* A program ...
	* consists of modules ...
	* that are made up of functions ...
	* that contain blocks ...
	* of instructions 
	* that operate on variables and data structures.

#### Modules
 * Developers (frequently) rely on libraries to build software. These libraries have well-documented functionality
 * Read the fine manual of the libraries, and focus your reversing effort on your actual target. 
	 * Most libraries are well-documented so you can tell what the target is trying to achieve with the library

#### Functions
* Functions represent fairly well-encapsulated functionality.
* Most functions have a well-defined goal such as:
	* set some data
	* calculate/retrieve/validate data 
	* dispatch other functions
	* perform some action on the outside world (via system calls)
* Initially functions can be reverse-engineered in isolation. Later, you can gain an understanding of how they fit together. 

#### Functions: The Control Flow Graph
* Functions are represented as a graph
* Each block is a set of instructions that will execute one after the other
* Blocks are joined by edges, representing conditional and unconditional jumps.


#### Functions Prologue:
* functions often begin with a **prologue** and end with an **epilogue** 
* Set up the **stack frame**
* Tear down the **stack frame**

#### The Stcak
* Recall ELF sections:
	* **.data** used for pre-initialized global writable data (such as global arrays with initial values)
	* **.rodata**: used for global read-only data (such as string constants)
	* **.bss**: used for uninitialized global writable data (such as global arrays without initial values)
	* Global stuff appear to live in the ELF
* But what about local variables?
* The stack is a region of memory used to store local variables and call contexts 

#### The Stack Grows Backwards
* Historical oddity: the stack grows backwards
* When you `push` to the stack, `rsp` is decreased by 8.
* When you `pop` from the stack, `rsp` is increased by 8.
* The stack is conceptualized vertically or horizontally.


#### The Stack: Initial Layout
* The stack starts out storing (among some other things) the environment variables and the program arguments. 
	* it starts out with these string tables (pointers to strings terminated by NULL)

#### The Stack: Calling a function
* When a function is called the address that the called function should return is implicity `push`ed onto the stack.
* This return address is implicitly `pop`ped when the function returns. 

#### The Stack: Function Frame Setup
* Every function sets up its stack frame. it has:
	* Stack pointer (`rsp`): points to the leftmost side of the stack frame
	* Base pointer (`rbp`): points to the rightmost side of the stack frame
* **Prologue**:
	1. Save off the caller's base pointer
	2. Set the current stack pointer as the base pointer
	3. "allocate" space on the stack (subtract from the stack pointer)

#### The Stack: Function Frame Teardown
* Every function sets up its stack frame. it has:
	* Stack pointer (`rsp`): points to the leftmost side of the stack frame
	* Base pointer (`rbp`): points to the rightmost side of the stack frame
* Epilogue:
	1. "deallocate" the stack (`mov rsp, rbp`)
		* a note that the data is NOT destroyed by default
	2. restore the old base pointer
* Now we are ready to return.


#### -fomit-frame-pointer
* Most compilers can omit frame pointers in most cases, to allow `rbp` to be used as a general purpose register. 

