# Chapter 1: UNIX System Overview
## Dated: 2020/11/25

### ls.c

```C
#include "apue.h"
#include <dirent.h>
#include <errno.h>		/* for definition of errno */
#include <stdarg.h>		/* ISO C variable aruments */
#include "error.h"

int main(int argc, char *argv[])
{
DIR
*dp;
struct dirent
*dirp;
if (argc != 2)
err_quit("usage: ls directory_name");
if ((dp = opendir(argv[1])) == NULL)
err_sys("can’t open %s", argv[1]);
while ((dirp = readdir(dp)) != NULL)
printf("%s\n", dirp->d_name);
closedir(dp);
exit(0);
}
```



New realization: you can run `cc -o ls ls.c` instead of gcc.

#### Pathname

A sequence of one or more filenames, separated by slashes and optionally starting with a slash, forms a pathname. A pathname that begins with a slash is called an absolute pathname; otherwise, it’s called a relative pathname. Relative pathnames refer to files relative to the current directory. The name for the root of the file system (/) is a special-case absolute pathname that has no filename component.

Get the UNIX Programmer's Manual.

On Linux find dirent.h: /usr/include/dirent.h has functions like functions opendir, readdir, and closedir. 

The opendir function returns a pointer to a DIR structure, and we pass this pointer to the readdir function. We don’t care what’s in the DIR structure. We then call readdir in a loop, to read each directory entry. The readdir function returns a pointer to a dirent structure or, when it’s finished with the directory, a null pointer. All we examine in the dirent structure is the name of each directory entry (d_name). Using this name, we could then call the stat function (Section 4.2) to determine all the attributes of the file.

two functions of our own to handle the errors: err_sys and err_quit can be found in the folder /headers/apue.3e/lib in a file called error.c. I found out about this from <https://stackoverflow.com/questions/24011090/where-is-err-quit-defined>. 

#### Working Directory

Every process has a working directory, sometimes called the current working directory. This is the directory from which all relative pathnames are interpreted. A process can change its working directory with the chdir function. For example, the relative pathname doc/memo/joe refers to the file or directory joe, in the directory memo, in the directory doc, which must be a directory within the working directory. From looking just at this pathname, we know that both doc and memo have to be directories, but we can’t tell whether joe is a file or a directory. The
pathname /usr/lib/lint is an absolute pathname that refers to the file or directory lint in the directory lib, in the directory usr, which is in the root directory.

#### Home Directory

When we log in, the working directory is set to our home directory. Our home directory
is obtained from our entry in the password file.

### copystdio.c

```c
#include "apue.h"
#include "error.h"

#define BUFFSIZE 4096

int main(void) {
	int n;
	char buf[BUFFSIZE];
	while ((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0 )
		if (write(STDOUT_FILENO, buf, n) != n)
			err_sys("write error");
	if (n < 0)
		err_sys("read error");
	exit(0);
}
```

copystdio.c works better than scanf for sure. Will have to find a write function that takes an argument for writing out to a variable.

`./copystdio <hello.txt> bye.txt`

#### Standard I/O

The standard I/O functions provide a buffered interface to the unbuffered I/O
functions. Using standard I/O relieves us from having to choose optimal buffer sizes, such as the BUFFSIZE constant in Figure 1.4. The standard I/O functions also simplify dealing with lines of input (a common occurrence in UNIX applications). The fgets function, for example, reads an entire line. The read function, in contrast, reads a specified number of bytes. As we shall see in Section 5.4, the standard I/O library provides functions that let us control the style of buffering used by the library. The most common standard I/O function is printf. In programs that call printf, we’ll always include <stdio.h>—normally by including apue.h—as this header contains the function prototypes for all the standard I/O functions.

### copystdio2.c

```C
#include "apue.h"

int main(void) {
	int c;
	while ((c = getc(stdin)) != EOF)
		if (putc(c, stdout) == EOF)
			err_sys("output error");
	if (ferror(stdin))
		err_sys("input error");
	exit(0);
}
```

The function getc reads one character at a time, and this character is written by putc. After the last byte of input has been read, getc returns the constant EOF (defined in <stdio.h>). The standard I/O constants stdin and stdout are also defined in the <stdio.h> header and refer to the standard input and standard output.

You can even run and look at the magic: `./copystdio2 <hello.txt> bye.txt`

```C
#include "apue.h"
int main(void)
{
	printf("hello world from process ID %ld\n", (long)getpid());
	exit(0);
}
```

#### Program

A program is an executable file residing on disk in a directory. A program is read into
memory and is executed by the kernel as a result of one of the seven exec functions.

#### Processes and Process ID

An executing instance of a program is called a process, a term used on almost every page of this text. Some operating systems use the term task to refer to a program that is being executed. The UNIX System guarantees that every process has a unique numeric identifier called the process ID. The process ID is always a non-negative integer.

When this program runs, it calls the function getpid to obtain its process ID. As we shall see later, getpid returns a pid_t data type. We don’t know its size; all we know is that the standards guarantee that it will fit in a long integer. Because we have to tell printf the size of each argument to be printed, we have to cast the value to the largest data type that it might use (in this case, a long integer). Although most process IDs will fit in an int, using a long promotes portability.

### commands.c

```C
#include "apue.h"
#include <sys/wait.h>
#include "error.h"

int main(void) {
	char buf[MAXLINE];
	pid_t pid;
	int status;

	printf("%% "); // (printf requires %% to print %)
	while (fgets(buf, MAXLINE, stdin) != NULL) {
		if (buf[strlen(buf) - 1] == ’\n’)
			buf[strlen(buf) - 1] = 0; /* replace newline with null */
		if ((pid = fork()) < 0) {
			err_sys("fork error");
		} else if (pid == 0) {	/* child */
			execlp(buf, buf, (char *)0);
			err_ret("couldn’t execute: %s", buf);
		exit(127);
		}
		/* parent */
		if ((pid = waitpid(pid, &status, 0)) < 0)
			err_sys("waitpid error");
		printf("%% ");}
	exit (0);
}
```

fgets reads the line 
