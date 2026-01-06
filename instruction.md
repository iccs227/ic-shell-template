# Overview

In this project, you will implement a simple shell for Linux. This shell will be called “icsh” or “IC shell”. The functionality of this shell will be similar to other popular Linux shells such as bash, csh, zsh, but with a subset of features. Basically, your icsh should have the following functionality:

* Interactive and batch mode
* Support built-in some commands
* Allow the user to execute one or more programs from executable files as either background or foreground jobs
* Provide job-control, including a job list and tools for changing the foreground/background status of currently running jobs and job suspension/continuation/termination.
* Allow for input and output redirection to/from files.

There are a number of things that you need to implement in this project. We organize the objectives into milestones below.

## Before you begin

To start the project, you should clone the starting repo and it should contain `icsh.c` and a `Makefile`. These two files will be your starting point. Make sure to use `make icsh` to generate the binary file called: `icsh`. We will ask you to `git tag` the commit for each of the milestones for grading. The tag name will be given in each milestone description below.

The look and feel of icsh should be similar to that of other UNIX shells, such as bash, tcsh, csh, etc. For example, your shell’s work loop should produce a prompt, e.g., `icsh $`, accept input from the user, and then produce another prompt. Messages should be written to the screen as necessary, and the prompt should be delayed when user input shouldn’t be accepted, as necessary. Needless to say, your shell should take appropriate action in response to the user’s input.

---

## Milestone 1: Interactive command-line interpreter

* **Git tag:** `0.1.0` (please tag the commit for this milestone)
* **Points:** 10

As your first milestone, you should implement a command-line interpreter functionality. Basically, when you start your shell with `icsh` with no argument, your shell should go into a prompt as shown below:

```bash
$ ./icsh
Starting IC shell
icsh $ <waiting for command>

```

You can design your own prompt. In the examples, here we use `icsh $` as our prompt symbol so you know we're in our shell. Optionally, you could write a welcome message when the shell starts.

For this milestone, your shell must support 3 built-in commands:

1. `echo <text>` -- the echo command prints a given text (until EOL) back to the console.
```bash
icsh $ echo hello world
hello world
icsh $

```


2. `!!` -- Double-bang; repeat the last command given to the shell.
```bash
icsh $ echo hello world
hello world
icsh $ !!
echo hello world
hello world
icsh $

```


Note that the command first prints out the last command first before running it. When there is no last command, `!!` just gives you back the prompt.
3. `exit <num>` -- this command exits the shell with a given exit code.
```bash
icsh $ exit 1
bye
$

```


Technically speaking, there are two approaches to do this. You can either return from the main function the exit code, or use a system call `exit()` (Links to an external site). The exit code should be in the range of 0-255. If a larger number is given, you must truncate them to fit in 8 bits. Optionally, a goodbye message can be shown on screen.

**Important notes:**

* When users give any other command, your shell should print out `bad command` and return to a prompt.
* When users give an empty command, your shell just give a new prompt.

---

## Milestone 2: Script mode

* **Git tag:** `0.2.0`
* **Points:** 10

For the milestone, you will enhance your shell to support script mode. When your shell is executed with an argument filename, your shell should read commands from the given file and run them one by one. For example,

**test.sh**

```bash
echo hello
echo world
!!
exit 5

```

Say we have `test.sh` with the above content in the current directory, you should be able to run it through your shell as follows.

```bash
$ ./icsh test.sh
hello
world
world
$ echo $?
5
$

```

Note the command `echo $?` in the example above, this command will print out the exit code of the previous command. Since we gave exit 5 to icsh, we expect the bash shell to report exit code = 5 here.

---

## Milestone 3: Running an external program in the foreground

* **Git tag:** `0.3.0`
* **Points:** 10

In this milestone, you will implement the most important functionality of a shell: running another program. Specifically, when user is given a command that doesn't match with any built-in command, it is assumed to be an external command. Your shell must spawn a new process, execute it and wait for the command to complete and resume control of the terminal. Here is an example of a user running `ls` command:

```bash
icsh $ ls
file_a file_b file_c
icsh $

```

An external command could also contain argument list. Your shell must support them.

```bash
icsh $ ls -la
total 8
drwxr-xr-x 2 tusr 1000 4096 May  2 07:04 .
drwxrwxrwt 1 root  root 4096 May  2 07:04 ..
-rw-r--r-- 1 tusr 1000    0 May  2 07:04 file_a
-rw-r--r-- 1 tusr 1000    0 May  2 07:04 file_b
-rw-r--r-- 1 tusr 1000    0 May  2 07:04 fille_c
icsh $

```

Check out the resources section below for how to create process / execute a program / wait for it to complete.

---

## Milestone 4: Signal Handler

* **Git tag:** `0.4.0`
* **Points:** 10

Through an interaction with the terminal driver, certain combinations of keystrokes will generate signals to your shell instead of appearing within stdin. Your shell should respond appropriately to these signals.

1. **Control-Z** generates a `SIGTSTP`. This should not cause your shell to be suspended. Instead, it should cause your shell to suspend the processes in the current foreground job. If there is no foreground job, it should have no effect.
2. **Control-C** generates a `SIGINT`. This should not kill your shell. Instead it should cause your shell to kill the processes in the current foreground job. If there is no foreground job, it should have no effect.

**Support additional build-in commands:**
`echo $?` -- this prints out the exit status code of the previous command. You may assume that all build-in commands exits with exit code 0

Check out the resources section below and read more about signal handling.

---

## Milestone 5: I/O redirection

* **Git tag:** `0.5.0`
* **Points:** 10

A program's execution may be followed by the meta-character `<` or `>` which is in turn followed by a file name. In the case of `<`, the input of the program will be redirected from the specified file name. In the case of `>`, the output of the program will be redirected to the specified file name. If the output file does not exist, it should be created. If the input file does not exist, this is an error.

For example,

```bash
icsh $ la -l > some_file

```

This above command starts a new process, executes ls, but instead of outputting to the STDOUT, the output is redirected to a file.

The goal for this milestone is the support both input redirection and output redirection. You can read about how to do this in the resource section below.

---

## Milestone 6: Background jobs and job control

* **Git tag:** `0.6.0`
* **Points:** 50

Each process executed by itself from the command line is called a **job**. When executing backgrounds jobs, the shell should not wait for the job to finish before prompting, reading, and processing the next command. When a background job finally terminates a message to that effect must be printed, by the shell, to the terminal. This message should be printed as soon as the job terminates.

To start a background job, user will issue a command and followed by `&` symbol. Here is an example:

```bash
1: icsh $ sleep 5 &
2: [1] 843
3: icsh $

```

**Explanation:**
Line 1, user issues a command "sleep 5" to be ran in the background (indicated by trailing &). The shell starts a new process, runs the command, prints out the job id and its PID to the console (Line 2) and returns to the prompt immediately (Line 3). This allows user to run another command while the sleep command is ran in the background.

```bash
icsh $
[1]+  Done                     sleep 5
icsh $

```

When the job is completed, the shell will asynchronously print out the job information stating the job id, Done status, the original command back to notify the user that one of the background has been completed.

As we have more than jobs running at a time, your shell must support the following commands for job control.

1. `jobs` - **list current jobs**
```bash
icsh $ sleep 100 &
[1] 855
icsh $ sleep 200 &
[2] 856
icsh $ jobs
[1]-  Running                 sleep 100 &
[2]+  Running                 sleep 200 &
icsh $

```


The `jobs` command shows a table of uncompleted job information (job ids, process status, commands).
2. `fg %<job_id>`: Brings the job identified by `<job_id>` into the foreground. If this job was previously stopped, it should now be running. Your shell should wait for a foreground child to terminate before returning a command prompt or taking any other action.
```bash
icsh $ sleep 100 &
[1] 862
icsh $ fg %1
sleep 100

```


`sleep 100` (job id=1) was started as a background job. Then, it was brought up to the foreground by the fg command. The shell is waiting for the command to complete.
3. `bg %<job_id>`: Execute the suspended job identified by `<job_id>` in the background.
```bash
icsh $ sleep 100
^Z
[1]+  Stopped                 sleep 100
icsh $ bg %1
[1]+ sleep 100 &
icsh $

```


This time `sleep 100` was started a a foreground job. While it is running, the user presses 'Ctrl+Z' to suspend it (SIGSTOP). The shell detects that, print out the status line. Then, the user gives the bg command to put job id =1 to resume executing in the background (SIGCONT) and the shell gives the prompt right away.

**Important notes:**
This milestone is the most complicated part of the project. Start early!

---

## Milestone 7: Extra features, Extra credits

* **Git tag:** `0.7.0`
* **Points:** 10

Congratulations for reaching this milestone. We know you're probably having a lot of fun writing your own shell. This part is for you to throw in any extra features for extra credits.
The points will be awarded proportional to how awesome your features are.

---

## Deliverables

We will have a demo session for you to demo your shell. In addition, you must submit a URL to your git repo on Canvas before the project deadline. In your Git repo, you must have:

* A `Makefile`.
* Source files that compile, by typing make, into an executable of name `icsh`.
* Tags of the milestones
* Optionally, a file of name `README` that contains anything you wish to point out to us.

---

## Resources

### System Calls

You have probably already heard the term “System Call.” Do you know what it means? As its name implies, a system call is a “call”, that is, a transfer of control from one instruction to a distant instruction. A system call is different from a regular procedure call in that the callee is executed in a privileged state, i.e, that the callee is within the operating system.

Because, for security and sanity, calls into the operating system must be carefully controlled, there is a well-defined and limited set of system calls. This restriction is enforced by the hardware through trap vectors: only those OS addresses entered, at boot time, into the trap (interrupt) vector are valid destinations of a system call. Thus, a system call is a call that trespasses a protection boundary in a controlled manner.

Since the process abstraction is maintained by the OS, icsh will need to make calls into the OS in order to control its child processes. These calls are system calls. In UNIX, you can distinguish system calls from user-level library (programmer's API) calls because system calls appear in section 2 of the `manual'', whereas user-level calls appear in section 3 of the `manual''. The `manual'' is, in UNIX, what you get when you use the `man'' command. For example, `man fork` will get you the `man page'' in section 2 of the manual that describes the `fork()` syscall, and `man -s 2 exec` will get you the `man page'' that describes the family of ``exec'' syscalls (a syscall, hence -s 2.)

The following UNIX syscalls may prove to be especially useful in your solution to this homework. There are plenty of others, so you may find “man” and good reference books useful, especially if you are new to system programming.

* **`pid_t fork(void)`:** It creates a process that is an almost-exact copy of the calling process; in particular, after a successful return from fork(), both parent and child processes are executing the same program. The two processes can be distinguished by the return value from fork().
* **`int execvp(const char * file, char * const argv[])`:** Loads the executable file path, or a file found through a search path, into the memory associated with the calling process, and starts executing the program therein. If successful, it obliterates whatever program is currently running in the calling process. There are several other, similar forms of exec.
* **`void exit(int status)`:** Exits the calling program, destroying the calling process. It returns status as the exit value to the parent, should the parent be interested. The parent receives this exit value through the wait syscall, below. Note that the linker introduces an exit() call at the end of every program, for instance, at the end of a C main procedure, even if the C code doesn't explicitly have one.
* **`pid_t wait(int *stat_loc)`:** Returns the exit status of an exited child, if any. Returns error if there are no children running. Blocks the calling process until a child exits if there are children but they are all currently running.
* **`pid_t waitpid(pid_t pid, int *stat_loc, int options)`:** Similar to wait() but allows you to wait for a specific process of group of processes, and allows the specification of flags such as WNOHANG.
* **`wait3(...)`, `wait4(...)`:** Similar to wait() but allow different combinations of parameters and flags.
* **`int tcsetpgrp(int fildes, pid_t pgid_id)`:** Sets the foreground process group id to be the foreground group associated with the controlling terminal. The controlling terminal is usually associated with stdin, stdout, and stderr (file descriptors 0, 1, and 2)
* **`int setpgid(pid_t pid, pid_t pgid)`:** Sets the process group ID of the process with ID pid to pgid.
* **`int dup2 (int filedes, int filedes2)`:** Causes the file descriptor filedes2 to refer to the same file as filedes.
* **`int pipe(int filedes[2])`:** Creates a pipe, placing the file descriptors into the supplied array of two filedescriptors.

### Process Creation

To create a new process we use the `fork()` system call. The fork system call actually clones the calling process, with very few differences. The clone has a different process id (PID) and parent process id (PPID). There are some other minor differences, see the man page for details.

The return value of the `fork()` is the only way that the process can tell if it is the parent or the child (the child is the new one). The fork returns the PID of the child to the parent and 0 to the child. This subtle difference allows the two separate processes to take two different paths, if necessary.

The `wait_()` family of functions allows a parent process to wait for a child process to complete. You may want to do this when you create a foreground process from your shell.

It is important to note that the `wait_()` family of functions returns any time the child changes status -- not just when it rolls over or exits. Many status changes you may want to ignore. You may also want to take a look at some of the flags in the man page for `waitpid()`, you may find `WNOHANG`, and others helpful. (WNOHANG makes the wait non-blocking, if there's no news -- it just lets you collect information, if available)

The following example shows a `waitpid()`. It waits for a specific child. `wait()` will wait for any child. There are several other flavors. We'll discuss more about what the `execve()` within the child does shortly.

```c
int main(int argc, char *argv[])
{
  int status;
  int pid;
  char *prog_argv[4];

  /* Build argument list */
  prog_argv[0] = "/usr/local/bin/ls";
  prog_argv[1] = "-l";
  prog_argv[2] = "/";
  prog_argv[3] = NULL;

  /*
   * Create a process space for the ls
   */

  if ((pid=fork()) < 0)
  {
    perror ("Fork failed");
    exit(errno);
  }
  if (!pid)
  {
    /* This is the child, so execute the ls */
    execvp (prog_argv[0], prog_argv);
  }

  if (pid)
  {
    /*
     * We're in the parent; let's wait for the child to finish
     */
    waitpid (pid, NULL, 0);
  }
}

```

It is important for your shells to wait for the children that they create. This can either be done in a blocking fashion for foreground processes, or in a non-blocking fashion (`WNOHANG`) when the child signals. Although many of the resources composing a process are freed when it dies, the **process control block (PCB)**, or at least some of its information, is not. The PCB contains status information that the parent can collect via `wait_()`. A process that is in this state is called **defunct**. After the `wait_()`, the PCB is freed. If the parent dies before the child, the child is reparented to the `init()` process which will perform a `wait_()` for any such process, allowing the PCB to be freed. Orphan process that are waiting for init to clean them up are called **zombies**.

### What If I Don't Want A Clone?

The `exec_()` family of calls allows a process to substitute another program for itself. Typically a program will call `fork()` to generate a duplicate copy of itself and the child will call an `exec_()` function to start another process.

There are several different flavors of `exec_()`. They all boil down to the same call within the kernel. One parameterization may be more or less convenient from time-to-time.

An exec'd process isn't completely different from the calling process. It does inherit some things, PPID, GID, and signal mask, but not signal handlers. Please see the man page for the details.

The `exec_()` functions do not return (a new process is now in charge). At least it is fair to say that if they do return, something bad has happened. The previous example code also illustrates `execvp()`.

### I/O Redirection

To implement I/O redirection, you'll need to use the `dup2()` function:

`int dup2(int fildes, int fildes2);`

Each process contains a table with one entry for each open file. This table contains some information about the state of the open file, such as the current offset into the file (the location where the next operation will occur). It also contains a pointer to the system-wide open file table.

This table contains exactly one entry for each open file in the system. If multiple processes have the same file open, the corresponding entry in each process's file descriptor table will point to the same entry in the system-wide open file table. This table contains some information about the file, including a count of how many processes currently have it open. It also contains a pointer to the file's inode, the data structure that associates a file with its physical storage on disk. We'll talk more about this when we get to file systems.

It is also important to realize that many non-files use the same interface, although they operate differently under the hood. For example, in many ways, terminals can be manipulated as if they were files. By default the first three entries in each process's open file table are open and reference the terminal: stdin (0), stdout(1), and stderr(2).

To perform I/O redirection, we open a file and then copy this file's file descriptor entry over either standard in or standard out (or standard error). If we need to restore the original entry later, we need to save it in another entry in the table.

The following is an example of I/O redirection.

```c
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <fcntl.h>

int main(int argc, char *argv[])
{
  int in;
  int out;
  size_t got;
  char buffer[1024];

  in = open (argv[1], O_RDONLY);
  out = open (argv[2], O_TRUNC | O_CREAT | O_WRONLY, 0666);

  if ((in <= 0) || (out <= 0))
  {
    fprintf (stderr, "Couldn't open a file\n");
    exit (errno);
  }

  dup2 (in, 0);
  dup2 (out, 1);

  close (in);
  close (out);

  while (1)
  {
    got = fread (buffer, 1, 1024, stdin);
    if (got <=0) break;
    fwrite (buffer, got, 1, stdout);
  }
}

```

### Signals

Signals are the simplest primitive for interprocess communication (IPC). Signals allow one process to communicate the occurrence of an event to another process. The number of the signal indicates which event occurred. No other information can be communicated via signals.

But signals will be very important in this project. They will indicate changes in the state of a child background process -- such as its termination, and other important events....that it’s time for a process to sleep, for example.

When a process receives a signal, it can take an action. Many signals have a default action. For example, certain signals, by default, cause core dumps, or process' to suspend themselves.

We can also specify how we want our process to handle a particular signal (Except for KILL, which isn't really a signal, although it looks like one to the programmer). We do this by specifying a signal handler.

The following is an example of a signal handler:

```c
/*
* This example shows a "signal action function"
* Send the child various signals and observe operation.
*
*/
void ChildHandler (int sig, siginfo_t *sip, void *notused)
{
  int status;

  printf ("The process generating the signal is PID: %d\n",
          sip->si_pid);
  fflush (stdout);

  status = 0;
  /* The WNOHANG flag means that if there's no news, we don't wait*/
  if (sip->si_pid == waitpid (sip->si_pid, &status, WNOHANG))
  {
    /* A SIGCHLD doesn't necessarily mean death - a quick check */
    if (WIFEXITED(status)|| WTERMSIG(status))
      printf ("The child is gone\n"); /* dead */
    else
      printf ("Uninteresting\n"); /* alive */
  }
  else
  {
    /* If there's no news, we're probably not interested, either */
    printf ("Uninteresting\n");
  }
}

int main()
{

  struct sigaction action;

  action.sa_sigaction = ChildHandler; /* Note use of sigaction, not
                                        handler */
  sigfillset (&action.sa_mask);
  action.sa_flags = SA_SIGINFO; /* Note flag,otherwise NULL in function*/
  sigaction (SIGCHLD, &action, NULL);

  fork();

  while (1)
  {
    printf ("PID: %d\n", getpid());
    sleep(1);
  }
}

```

### Process Groups, Sessions, and Process Groups, Sessions, and Job Control

When we log into a system, the operating system allocates a terminal for our session. A session is an environment for processes that is (or at least can be) associated with one controlling terminal.

Our shell is placed into the **foreground process group** within this session. A process group is a collection of one process or of related processes -- they are usually related by one or more pipes. At most, one terminal can be associated with a process group. The **foreground** process group is the group within a session that currently has access to the controlling terminal. Since there is only one controlling terminal per session, there can only be one foreground process group.

Processes in the foreground process group have access to the stdin and stdout associated with the terminal. It also means that certain key combinations, cause the terminal driver to send signals to all processes in the foreground process group. In the case of CONTROL-C, SIGINT is sent to each process. In the case of CONTROL-Z, SIGTSTP is sent to each process. These key combinations do not result in character being placed in stdin.

There can also be background process groups. These are process groups that do not currently have access to the sessions controlling terminal. Since they don’t have access to the controlling terminal, they can't perform terminal I/O to/from the controlling terminal. If a background process tries to interact with the controlling terminal, it is sent a SIGTTOU or SIGTTIN, as appropriate. By default, these signals act like a SIGTSTP and suspend the process. The parent process (the shell) is notified about this change, much like it would be if the child process received a SIGTSTP, died, &c. It can discover these changes through the status returned by `wait()`. Your shell will have to handle these changes in its children.

Processes are placed into process groups using the `setpgid()` function. Process groups are named by the PID of the **group leader**. The group leader is the first process to create a group -- it's PID becomes the GID. The group leader can die and the group can remain.

A group becomes the foreground group using the `tcsetpgrp()` call. This call makes the specified group the foreground group. It can affect itself or any of its children.

If a process forms a new session by calling `setsid()`, it becomes both a session leader and a group leader. For a new session to interact with a terminal, it must allocate a new one -- you won't need to create a new session or allocate a terminal. Instead, exec (“exec icsh”) your icsh from the login shell (csh, sh, bash, csh, etc). This will replace the original shell with your shell, making your shell the only process in the foreground process group.

You will have to create process groups, and manipulate the foreground process group to make sure the right process group is the current foreground process (which could be your shell).

By making the right group the foreground process group, you are not only ensuring that it has a connection to the terminal for stdin, stdout, and stderror, but you are also ensuring that every process in the foreground group will receive terminal control signals like SIGTSTP.

Please remember that you have a choice in this homework – you can take the short-cut. But either way, we’d like you to understand how this works. If you do take the shortcut, you can leave all of the child processes in the same group as the shell and masked SIGTSTP when they are created, so that only the shell can recieve it. The shell can then propogate the equivalent (but unmaskable) SIGSTOP to the appropriate children. The approach falls apart, for example, if you want to try to run your shell form within your shell – but it is good enough for this homework.

Here’s an example that illustrates `tcsetgrp()` and `setpgrp()`:

```c
#include <stdio.h>
#include <signal.h>
#include <stddef.h>
#include <sys/wait.h>
#include <sys/ioctl.h>
#include <sys/termios.h>

/* NOTE: This example illustrates tcsetgrp() and setpgrp(), but doesn’t function correctly because SIGTTIN and SIGTTOU aren’t handled.*/

int main()
{
  int status;
  int cpid;
  int ppid;
  char buf[256];
  sigset_t blocked;

  ppid = getpid();


  if (!(cpid=fork()))
  {
    setpgid(0,0);
    tcsetpgrp (0, getpid());
    execl ("/bin/vi", "vi", NULL);
    exit (-1);
  }

  if (cpid < 0)
    exit(-1);


  setpgid(cpid, cpid);
  tcsetpgrp (0, cpid);

  waitpid (cpid, NULL, 0);

  tcsetpgrp (0, ppid);

  while (1)
  {
    memset (buf, 0, 256);
    fgets (buf, 256, stdin);
    puts ("ECHO: ");
    puts (buf);
    puts ("\n");
  }
}

```
