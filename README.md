# PROJECT WEBSERV FOR 42
By chdonnat (Christophe Donnat from 42 Perpignan, France)

## AIM OF THE PROJECT:



### BONUS PART:



## SOME COMMANDS YOU CAN USE:

compile the program and suppress the .o files:

	make && make clean

execute the program

	./webserv <configuration_file>

 execute the program with valgrind

	make val

## ARCHITECTURE:
- minishell/ directory with the whole project
	- libft/ directory with the libft (+ get_next_line and ft_printf)
 	- dclst/ directory with functions and header for using doubly circular linked list
	- bonus/ directory with files for the bonus part
		- src/ directory containing the main files of the project
  			- builtins/ files for builtins commands
     			- env/ files with functions needed to interact with environment variables
        		- executor/ files to execute the command line
          		- lexer/ files for splitting the user input into tokens, store them in a chained list, check the syntax and create a binary tree
            		- signals/ files for handling signals
          		- text_transformer/ files for managing '$', '*' and '~'
		- utils/ directory for secondary files
		- include/ directory for headers
	- mandatory/ directory for the mandatory part (empty - everything is in the bonus directory)
- Makefile (with rules: make bonus clean fclean re)
- readme.md for quick explanation and main commands of the project
- valgrind.sup is a file containig a list of readline() leaks to suppress when executing valgring

## ABOUT MY PROJECT:



## DOCUMENTATION


---

### `execve()`

```cpp
#include <unistd.h>

int execve(const char *pathname, char *const argv[], char *const envp[]);
```

* **pathname**: Path to the executable file.
* **argv**: Null-terminated array of argument strings.
* **envp**: Null-terminated array of environment variable strings.

**Returns**:
Never returns on success (the current process is replaced). Returns -1 on error, and sets `errno`.

The `execve()` function replaces the current process image with a new process image specified by `pathname`. It is the low-level system call behind the `exec()` family. Upon success, the calling process becomes the new program and does not return.

### Key Use Cases

* Launching a CGI script in webserv.
* Replacing the current process with a new binary (e.g., `/usr/bin/php`).
* Manual process control in forked children.

### How It Works

`execve()` does not create a new process. It **replaces** the current process memory, stack, and code with the target program. You typically call it **after a `fork()`** to execute a new process in the child.

### Example Usage

```cpp
#include <unistd.h>

int main()
{
    char *args[] = {"/bin/ls", "-l", NULL};
    char *env[] = {NULL};

    execve("/bin/ls", args, env);
    // If execve fails
    perror("execve failed");
    return 1;
}
```

### Error Handling

Returns `-1` on failure, sets `errno`:

* `ENOENT`: File not found
* `EACCES`: Permission denied
* `ENOMEM`: Not enough memory
* `EFAULT`: Invalid address

### In Webserv

In Webserv, you will use `execve()` to:
✅ **Launch CGI scripts** (e.g., Python, PHP) in child processes
✅ **Replace the child process image** with a new executable
✅ **Pass HTTP environment variables** to the CGI via `envp` array

---

### `dup()`

```cpp
#include <unistd.h>

int dup(int oldfd);
```

* **oldfd**: File descriptor to duplicate.

**Returns**: A new file descriptor on success, or -1 on error.

`dup()` creates a copy of an existing file descriptor. The copy refers to the same underlying file or socket, sharing the same file offset and flags.

### Key Use Cases

* Redirect output/input streams (e.g., for CGI or logging).
* Temporarily replace `stdin`/`stdout`/`stderr`.

### How It Works

The new descriptor is returned with the **lowest available number**. Both descriptors can be used interchangeably.

### Example Usage

```cpp
#include <unistd.h>
#include <fcntl.h>

int main()
{
    int fd = open("log.txt", O_WRONLY | O_CREAT, 0644);
    int backup = dup(STDOUT_FILENO);
    dup2(fd, STDOUT_FILENO);
    write(STDOUT_FILENO, "Hello!\n", 7);
    dup2(backup, STDOUT_FILENO);
    return 0;
}
```

### Error Handling

Returns `-1` on error, sets `errno`:

* `EBADF`: Invalid file descriptor
* `EMFILE`: Too many file descriptors

### In Webserv

In Webserv, you will use `dup()` to:
✅ **Backup original file descriptors** before redirection
✅ **Restore original `stdout`/`stdin`** after executing a CGI
✅ **Preserve system-level streams during file/socket manipulation**

---

### `dup2()`

```cpp
#include <unistd.h>

int dup2(int oldfd, int newfd);
```

* **oldfd**: Existing file descriptor.
* **newfd**: Target descriptor to overwrite or create.

**Returns**: `newfd` on success, or -1 on error.

`dup2()` duplicates a file descriptor to a specific target number, closing `newfd` first if necessary.

### Key Use Cases

* Overwriting `stdout` or `stdin` with another descriptor.
* Setting up pipes between processes.

### How It Works

If `newfd` is open, it's closed first. If `oldfd == newfd`, `dup2()` returns immediately with no effect.

### Example Usage

```cpp
#include <unistd.h>
#include <fcntl.h>

int main()
{
    int fd = open("output.txt", O_CREAT | O_WRONLY, 0644);
    dup2(fd, STDOUT_FILENO); // Now stdout points to file
    write(STDOUT_FILENO, "File output\n", 12);
    return 0;
}
```

### Error Handling

Returns `-1` on error, sets `errno`:

* `EBADF`: Invalid descriptor
* `EMFILE`: Too many open files

### In Webserv

In Webserv, you will use `dup2()` to:
✅ **Redirect `stdin`/`stdout`** for CGI scripts
✅ **Connect pipe ends to `stdin` or `stdout`**
✅ **Overwrite file descriptors cleanly for execution**

---

### `pipe()`

```cpp
#include <unistd.h>

int pipe(int pipefd[2]);
```

* **pipefd**: Array of two integers. `pipefd[0]` is for reading, `pipefd[1]` for writing.

**Returns**: `0` on success, or `-1` on error.

`pipe()` creates a unidirectional data channel between two file descriptors. It’s commonly used for inter-process communication (IPC).

### Key Use Cases

* Set up communication between parent and child processes.
* Redirect output from a CGI or child to the parent.

### How It Works

After a `pipe()` call:

* Data written to `pipefd[1]` can be read from `pipefd[0]`
* Requires a `fork()` to set up separate processes using opposite ends

### Example Usage

```cpp
#include <unistd.h>
#include <stdio.h>

int main()
{
    int pipefd[2];
    char buffer[128];

    pipe(pipefd);
    write(pipefd[1], "hello", 5);
    read(pipefd[0], buffer, 5);
    buffer[5] = '\0';
    printf("Read: %s\n", buffer);
    return 0;
}
```

### Error Handling

Returns `-1` on error:

* `EMFILE`: Too many descriptors
* `EFAULT`: Invalid address

### In Webserv

In Webserv, you will use `pipe()` to:
✅ **Connect parent and CGI child processes**
✅ **Redirect CGI output to Webserv**
✅ **Read CGI script output without using intermediate files**

---

### `strerror()`

```cpp
#include <cstring>

char *strerror(int errnum);
```

* **errnum**: Error code (usually from `errno`).

**Returns**: Pointer to a human-readable string describing the error.

`strerror()` converts an error number (e.g., `errno`) into a readable message.

### Key Use Cases

* Display user-friendly error messages.
* Log system errors in your server.

### Example Usage

```cpp
#include <iostream>
#include <cstring>
#include <cerrno>

int main()
{
    std::cerr << strerror(ENOENT) << std::endl;
    return 0;
}
```

### In Webserv

In Webserv, you will use `strerror()` to:
✅ **Log system call errors (e.g., open, socket)**
✅ **Debug unexpected behavior by printing meaningful messages**
✅ **Improve error messages for better maintainability**

---

### `gai_strerror()`

```cpp
#include <netdb.h>

const char *gai_strerror(int errcode);
```

* **errcode**: Error code returned by `getaddrinfo()`.

**Returns**: Human-readable string explaining the `getaddrinfo()` error.

`gai_strerror()` is the counterpart of `strerror()` but for DNS/network-related errors from `getaddrinfo()`.

### Key Use Cases

* Diagnose `getaddrinfo()` failures.
* Log networking setup issues.

### Example Usage

```cpp
#include <iostream>
#include <netdb.h>

int main()
{
    std::cerr << gai_strerror(EAI_AGAIN) << std::endl;
    return 0;
}
```

### In Webserv

In Webserv, you will use `gai_strerror()` to:
✅ **Explain DNS resolution errors**
✅ **Log or display informative messages when `getaddrinfo()` fails**
✅ **Improve debugging when setting up sockets**

---

### `errno`

```cpp
#include <cerrno>
```

* `errno` is a global variable set by system calls and some library functions in case of error.

**Type**: `int`, declared as `extern int errno`.

### How It Works

After a system call or library function fails, `errno` is set to indicate the error. Use `errno` immediately after a failing call, as it may be overwritten.

### Common Values

* `ENOENT`: No such file or directory
* `EACCES`: Permission denied
* `ENOMEM`: Out of memory
* `EPIPE`: Broken pipe

### Example Usage

```cpp
#include <iostream>
#include <unistd.h>
#include <cerrno>
#include <cstring>

int main()
{
    if (access("nonexistent", F_OK) == -1)
        std::cerr << "Error: " << strerror(errno) << std::endl;
    return 0;
}
```

### In Webserv

In Webserv, you will use `errno` to:
✅ **Capture and interpret system errors after failures**
✅ **Use with `strerror()` or `gai_strerror()` to produce readable error logs**
✅ **Handle specific errors like `EPIPE` for broken connections or `EAGAIN` for non-blocking I/O**

---

Souhaites-tu que je t’en prépare d’autres (par exemple `socket()`, `bind()`, `listen()`, etc.) ?

