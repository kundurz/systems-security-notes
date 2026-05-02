# Static Linking
* Background information: compiler usually searches for header files in `/usr/local/include` and `/usr/include`

* This is referred to as "static linking" because it can't change once it's been done.
#### Sharing Code
* Perhaps you want to write code that will be available to lots of programs in different folders and you don't want a separate copy of the security code for each one.
	* There are 2 sets of files you want to share between programs: the `.h` header files and the `.o` object files.

#### Sharing Headers
Here are the main ways:
1. Store them in a standard directory like `/usr/local/include`
2. Put the full pathname in your include statement
3. Tell the compiler where to find the header files.

#### Sharing Object Files
* Put the object files in a shared directory and copy the full path whenever you use a compile command.

The above is fine for 1 or 2 object files, but it can become tedious when there are several object files.
#### Archives
* If you create an archive of object files, you can tell the compiler about a whole set of object files at once.
* This makes it much easier to share code between projects.
* `.a` files are a type of file containing other files.
* You can use the `nm` command to look inside archives
	* Lists names that are stored inside the archive.
* Different platforms can have slightly different archive formats.
* In an archive, the object files are NOT linked together like an executable, they are stored in the archive as distinct files.

#### Creating Archives
* Use the `ar` command.
* Ex: `ar -rcs libhfsecurity.a encrypt.o checksum.o`
	* `-r`: the .a file will be updated if it already exists
	* `-c`: the archive will be created without any feedback
	* `-s`: tells ar to create an index at the start of the file.
* Naming convention: `lib<something>.a`

#### Compiling with Archives
* Use the `-l<something>` flag to tell the compiler to look for a library `libhfsecurity`
* Ex: `gcc test_code.c -lhfsecurity -o test_code`
	* This is assuming `libhfsecurity.a` is in a system directory
	* If you put the archive somewhere else, you'll have to account for that with the `-L` flag:
		* `gcc test_code.c -L/my_lib -lhfsecurity -o test_code`

# Dynamic Linking

#### Linker
* Stitches pieces of compiled code together.

#### Limitations of Static Linking
* Once files are linked, you can't change them. 
* You really have no way of changing any of the ingredients without rebuilding the whole program.

#### Dynamic Linking
* What we want is interchangeable pieces of code.
* Allows you tp update an application without needing to recompile it. 
* The reason you can't change different pieces of object code is because they are all contained in a single file. They were *statically linked* together when the program was compiled.
* If your program was instead made up of lots of separate files that only joined together when the program runs, you would avoid this problem.

#### Linking .a at runtime
* It seems like you have `.a` files which are files containing object code. 
* Unfortunately, simple object files and archives don't have quite enough information in them to be linked together at runtime. 

#### Dynamic Libraries
* Very similar to object files, but not quite the same. 
* Unlike an archive, the object files are properly linked together in a dynamic library to form a single piece of object code.
* Contains extra information that the operating system will need to link the library to other things.

#### Creating a dynamic library
 * Compile your `.c` code into object code as usual, but this tells `gcc` you want to create position-independent code. 
 * Some operating systems and processors need to build libraries from position-independent code so they can decide at runtime where they want to load it in memory
 * Use `gcc -shared object_file.o /libs/libname.o` to convert the position-independent object code into a dynamic library.
 * IMPORTANT: When the compiler creates a dynamic library, it remembers the name it was created with, so you can't just rename the file afterwards, if you need to rename a library you must recompile it with a new name.
 * You compile the main program with a dynamic library similarly to how you did with the static one.

#### Compiling with a dynamic library
* The compiler won't include the library code into the executable file. Instead, it will insert some placeholder code that will track down the library and link to it at runtime. 
* On linux, ensure your library directory is added to the `LD_LIBRARY_PATH` and if you make sure you export it, then the main program can find the path.

#### Position-Independent code
* Code that can be loaded anywhere in memory.
* Some operating systems use a technique called memory mapping when loading dynamic libraries, which means all code is effectively position independent. 
* If you compile code on windows, you might get a warning that the `-fPIC` option is not needed. 

#### Naming Conventions
* Dynamic libraries on most operating systems are essentially the same thing, but what they're called can vary a lot
	* Windows: `.dll`
	* Linux `.so`
	* Mac: `.dylib`

#### Memory use
* Some operating systems will load separate copies of shared libraries for each process, others load shared copies to save memory.


