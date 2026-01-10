# Linux Process Loading

#### Introduction
1. Process is created
2. Process is loaded
3. Process is initialized

#### Portrait of a process
* Every linux process has:
	* state (running, waiting, stopped, zombie)
	* priority (and other scheduling information)
	* Parent, siblings, children
	* shared resources (files, pipes, sockets)
	* virtual memory space
	* security context
		* effective uid and gid
		* saved uid and gid
		* capabilities.


#### Where do processes come from
* processes propagate by mitosis!
* `fork` and (more recently) `clone` system calls that create a nearly exact copy of the calling process: a *parent* and a *child* 
* Later, the child process usually uses the `execve` syscall to replace itself with another process.

#### Can we load?
* Before anything is loaded, the kernel checks for executable permissions. If a file is not executable, `execve` will fail.

#### What to load?
* To figure out what to load, the Linux kernel reads the beginning of the file (i.e., `/bin/cat`), and make a decision:
	1. If the file starts with `#!`, the kernel extracts the interpreter from the rest of that line and executes this interpreter with the original file (the one you wrote) as an argument. (This is shell scripting).
	2. If the file matches a format in `/proc/sys/fs/binfmt_misc`, the kernel executes the interpreter specified for that format with the original file as an argument.
		* It may have to have a specific set of bytes at the beginning or may have to be put through a specific mask that specifies what needs to match and what doesn't need to match.
	3. (what if it does not match a format in the previous step) If the file is a dynamically-linked ELF, the kernel reads the interpreter/loader defined in the ELF, loads the interpreter and the original file, and lets the interpreter take control. 
	4. If the file is a statically-linked ELF, the kernel wil load it
	5. Other legacy file formats are checked for
* These can be recursive!


#### Dynamically linked ELFs: the interpreter
* process is done by the ELF interpreter specified in the binary
	* `readelf -a /bin/cat | grep interpret`
	* Colloquially known as the "loader"
* Can be overridden: `/lib64/ld-linux-x86-64.so.2 /bin/cat /flag`
* This is the runtime dynamic linker, not a traditional interpreter like Python or Bash
* it "interprets" in the sense that it resolves and links shared libraries at runtime, which is more than just loading memory. It also:
	* Applies relocations
	* Resolves symbols
	* Sets up the GOT/PLT
	* Prepares everything so that `main()` in your executable can run correctly. 

#### Dynamically linked ELFs: the loading process
1. The program and its interpreter are loaded by the kernel.
2. The interpreter locates the libraries (there is a loading order)
	a. `LD_PRELOAD` environment variable, and anything in /etc/ld.so preload
	* You can override functions in future loaded libraries. 
	b.`LD_LIBRARY_PATH` environment variable (can be set to the shell)
	* It will try to load everything here first. 
	c. `DT_RUNPATH` or `DT_RPATH` specified in the binary (both can be modified with patchelf).
	d. system-wide configuration (`/etc/ld.so.conf`)
	e. `/lib` and `/usr/lib`
3. The interpreter loads the libraries
	a. These libraries can depend on other libraries, causing more to be loaded
	b. relocations updated. 
* Priority:
	* `LD_PRELOAD`: If a symbol is found here, it's used regardless of what's in any othe rlocation. Used to override functions in shared libraries -- useful for debugging, patching, etc. If a match is found, the search stops. 
		* Primarily a developer/debugging tool, not something used by the system in normal operations. 
	* `DT_RPATH` (depreciated): Comes from the ELF binary itself. 
	* `LD_LIBRARY_PATH`: An environment variable specifying directories to search for shared libraries. Checked after `LD_PRELOAD` and before system defaults, unless `DT_RUNPATH` is present, which changes this order. 
	* `DT_RUNPATH`:
		* Also embedded in ELF binary. 
		* Overrrides `LD_LIBRARY_PATH` if present. 
	* Default system paths: `/lib`, `/usr/lib`, `/lib64` 

#### Where is this all getting loaded to?
* Each linux process has *virtual memory* space. It contains:
	* The binary
	* The libraries
	* The "heap" (for dynamically allocated memory)
	* The "stack" (for function local variables)
	* Any memory specifically mapped to the program
	* some helper regions
	* kernel code in the "upper half" of memory (above 0x8000000000 on 64-bit architectures), inaccessibelt to the process. 
* Virtual memory is dedicated to your proces.
* Physical memory is shared among the whole system.
* You can see the whole space by looking at `/proc/self/maps`

#### The Standard C Library
* `libc.so` is linked by almost every process.
* Provides functionality you take for granted.
	* `printf()`
	* `scanf()`
	* `socket()`
	* `atoi()`
	* `malloc()`
	* `free()`

#### The loading process (for statically linked binaries)
1. The binary is loaded

#### Process is initialzied
* Every ELF binary can specify constructors, which are functions that run before the program is actually launched
* For example, depending on the version, libc can initialize memory regions for dynamic allocations (malloc/free) when the program is launched
* You can specify your own
```c
__attribute__((constructor)) void haha()
{
	write(1, "haha!", 6); 
}
```
