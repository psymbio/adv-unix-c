New realization: you can run `cc -o ls ls.c` instead of gcc.

## Pathname

A sequence of one or more filenames, separated by slashes and optionally starting with a slash, forms a pathname. A pathname that begins with a slash is called an absolute pathname; otherwise, it’s called a relative pathname. Relative pathnames refer to files relative to the current directory. The name for the root of the file system (/) is a special-case absolute pathname that has no filename component.

Get the UNIX Programmer's Manual.

On Linux find dirent.h: /usr/include/dirent.h has functions like functions opendir, readdir, and closedir. 

The opendir function returns a pointer to a DIR structure, and we pass this pointer to the readdir function. We don’t care what’s in the DIR structure. We then call readdir in a loop, to read each directory entry. The readdir function returns a pointer to a dirent structure or, when it’s finished with the directory, a null pointer. All we examine in the dirent structure is the name of each directory entry (d_name). Using this name, we could then call the stat function (Section 4.2) to determine all the attributes of the file.

two functions of our own to handle the errors: err_sys and err_quit can be found in the folder /headers/apue.3e/lib in a file called error.c. I found out about this from <https://stackoverflow.com/questions/24011090/where-is-err-quit-defined>. 

## Working Directory
Every process has a working directory, sometimes called the current working directory. This is the directory from which all relative pathnames are interpreted. A process can change its working directory with the chdir function. For example, the relative pathname doc/memo/joe refers to the file or directory joe, in the directory memo, in the directory doc, which must be a directory within the working directory. From looking just at this pathname, we know that both doc and memo have to be directories, but we can’t tell whether joe is a file or a directory. The
pathname /usr/lib/lint is an absolute pathname that refers to the file or directory lint in the directory lib, in the directory usr, which is in the root directory.



## Home Directory

When we log in, the working directory is set to our home directory. Our home directory
is obtained from our entry in the password file.



copystdio.c works better than scanf for sure. Will have to find a write function that takes an argument for writing out to a variable.