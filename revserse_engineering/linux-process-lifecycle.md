# Linux Process Lifecycle 

#### 1. Cat is launched
* A normal ELF automatically calls `__libc_start_main()` in libc, which in turn calls the program's main function
	* In `_start` there is usually a bunch of setup, and then `__libc_start_main()` is called
	* For a number of reasons, this is just how it is.

#### 2. Cat reads its arguments and environment from memory
* `int main(int argc, void **argv, void **envp);`
	* You can get environment variables from using this envp syntax. 
* Your process' entire input from the outside world, at launch 
* An environment variable is an extra argument that you can pass outside of the argv set of strings to your program.
* There are many environment variables set. Type `env` to see them. 
* Your proces' entire input from the outside world at launch, comprises of:
	* The loaded objects (binaries and libraries)
	* command-line arguments in argv
	* "environment" in envp. 

#### 3. Cat does its thing (using library functions).
* The binary's import symbols have to be resolved using the libraries' export symbols.
* In the past, this was an on-demand process and carried great peril.
* In modern times, this is all done when the binary is loaded and is much safer.
* Use `nm -D` to see the various library functions (symbols) that a binary imports.
* Use `nm -a` to see the symbols that are exported. 
* Dynamically linked binaries are libraries of their own. 

#### 3. Interacting with the environment
* Almost all programs have to interact with the outside world.
* This is primarily done via *system calls* (`man syscalls`). 
* Each system call is well documented in section 2 of the main pages
* We can trace process system calls using `strace`
* "system calls" in C are actually `libc` functions. They are usually wrappers that trigger the correct system call. 
* System calls have very well-defined interfaces that very rarely change
* There are over 300 syscalls in linux

#### 3. Signals
* System calls are a way for a process to call into the OS. What about the other way around?
* Signals are how the OS talks to your process.
* Signals pause process execution and invoke the handler for that particular signal.
* Handlers are functions that take one single argument, the signal number.
* Without a handler for a signal, the default action is used (often, kill). 
* `SIGNKILL` (signal 9) and `SIGSTOP` (signal 19) cannot be handled)
* `pgrep` gives you a process' PID(s)

#### 3. Shared memory
* Another way of interacting with the outside world is by sharing memory with other processes.
* Requires system calls to establish, but once established, communication happens without system calls
* Easy way: use shared memory-mapped file in `/dev/shm` 

#### 4. Process Termination
* Process terminate by one of two ways:
	1. Recieving an unhandled singal.
	2. Calling the exit() system call.
* All processed must be "reaped":
	* After termination they will remain a zombie state until they are `wait()`ed on by their parent. 
	* When this happens, their exit code will be returned to their parent, and the process will be freed
	* if their parent dies without `wait()`ing on them, the are re-parented to PID 1 and will stay there until they're cleaned up.
