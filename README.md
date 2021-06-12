# Logging tool for command-line terminal and batch

## Introduction

This shell script `log4t` permits to log to files every text written to standard output and standard error streams by the execution of a batch or a command-line into a terminal, as well as text sent to the standard input stream.

> `log4t` is written in kornshell (compatible with `ksh` and `ksh93`). Don't hesitate to port it to `bash` if you know about bash coprocesses (and let me know !).

For instance, you can log the result of the command `ls README.md toto` with:

```
> ./log4t ls README.md toto
(+0) [ls] L ----- START execution of 'ls README.md toto' (process 6437 submitted by laurent@amedea6:/home/laurent/Informatique/Ksh/Sources/log4t)
(+0) [ls] L ----- Log file:   /home/laurent/Informatique/Ksh/Sources/log4t/ls.20210612T210810.6437.log
(+0) [ls] L ----- Error file: /home/laurent/Informatique/Ksh/Sources/log4t/ls.20210612T210810.6437.err
(+0) [ls] E ls: cannot access 'toto': No such file or directory
(+0) [ls] O README.md
(+0) [ls] L ----- FATAL ERROR (return code 2)
(+0) [ls] L ----- END execution of 'ls README.md toto' after 0.014 seconds (return code 2)
(+0) [ls] L ----- Log file:   /home/laurent/Informatique/Ksh/Sources/log4t/ls.20210612T210810.6437.log
(+0) [ls] L ----- Error file: /home/laurent/Informatique/Ksh/Sources/log4t/ls.20210612T210810.6437.err
```

Traces of execution are logged in a default log file.
Errors are sent a separate log file.

```
> cat ls.20210612T210810.6437.log
2021-06-12T21:08:10 [ls] L ----- START execution of 'ls README.md toto' (process 6437 submitted by laurent@amedea6:/home/laurent/Informatique/Ksh/Sources/log4t)
2021-06-12T21:08:10 [ls] L ----- Error file: /home/laurent/Informatique/Ksh/Sources/log4t/ls.20210612T210810.6437.err
2021-06-12T21:08:10 [ls] O README.md
2021-06-12T21:08:10 [ls] L ----- FATAL ERROR (return code 2)
2021-06-12T21:08:10 [ls] E ls: cannot access 'toto': No such file or directory
2021-06-12T21:08:10 [ls] L ----- END execution of 'ls README.md toto' after 0.014 seconds (return code 2)
2021-06-12T21:08:10 [ls] L ----- Error file: /home/laurent/Informatique/Ksh/Sources/log4t/ls.20210612T210810.6437.err
```

```
> cat ls.20210612T210810.6437.err
2021-06-12T21:08:10 [ls] L ----- START execution of 'ls README.md toto' (process 6437 submitted by laurent@amedea6:/home/laurent/Informatique/Ksh/Sources/log4t)
2021-06-12T21:08:10 [ls] E ls: cannot access 'toto': No such file or directory
2021-06-12T21:08:10 [ls] L ----- FATAL ERROR (return code 2)
2021-06-12T21:08:10 [ls] L ----- END execution of 'ls README.md toto' after 0.014 seconds (return code 2)
```

## Help page

> Type `log4t -h` for options.

### Name:

   `log4t` - executes a command and duplicates its terminal outputs to log files

### Usage:
   `log4t [-ahnrl] [{-|+}IOE] [-o filename] command [arguments...]`

### Description:

   The `log4t` command makes time-stamped typescripts of everything read from standard input and displayed to standard output and standard error during the execution of *command*.
   
   Typescripts are written to two log files, a standard log file (suffixed `.log`) and an error log file (suffixed `.err`).

   1. The command *command* is executed. *command* must be executable.
   2. The standard output (file descriptor 1) of "command" is:
      - typescripted to the standard log file if the standard output is associated with a terminal device or option +O is used.
      - typescripted to the standard output if it is associated to a terminal device. Otherwise, the standard output is kept unchanged.
   3. The standard input (file descriptor 0) of *command* is:
      - typescripted to the standard log file if the standard input is associated with a terminal device or option +I is used.
      - typescripted to the standard error if the standard error is associated to a terminal device.
   4. The standard error (file descriptor 2) of *command* is:
      - typescripted to the standard log file and the error log file if the standard error is associated with a terminal device or option +E is used.
      - typescripted to standard error if it is associated to a terminal device. Otherwise, the standard error is kept unchanged.
   5. The return status of the invoked *command* is returned.
   6. The `SIGHUP` signals are ignored during execution of *command*, allowing the user to logout before the execution ends (same as `nohup`) when run in background.

### Options:

   -a               Appends the typescripts to the log and error files.
   
   -r               Log and error files are overwritten.
   
   -o *filename*      If *filename* is an existing directory, specifies the directory where log files are written into.
                    Otherwise, specifies the basename for log files.

   -n               No log and error files are created.

   -h               Displays this help message.
   
   -l (lowercase ell)         Typescript written with long format (process id and line numbering).
   
   {-|+}I (uppercase i)          Desactivate/activate logging for standard input.
                    -I is needed if log4t is run as a background process (to avoid signal `TTIN`).
                    
   {-|+}O           Desactivate/activate logging for standard output.
   
   {-|+}E           Desactivate/activate logging for standard error.

### Names of log files:
   * Log files are written into the current directory, except if options -o is used and specifies an existing directory.
   * New output log files are created, except if options -a, -r or -n are used.
   * The basename of log files can be specified with option -o *filename*. By default, *command* is used as basename.
   * If options -a or -r are used, filenames are `basename.log` and `basename.err` for standard and error log files respectively.
   * Otherwise, filenames are `basename.date.pid.log` and `basename.date.pid.err` for standard and error log files respectively.

### Typescripting format:
   * to terminal : (+S) [L] T message
   * to log file (short format) : D [L] T message
   * to log file (long format, option -l) : D (+ss) P [L] T:N message
     where:
     - S is the number of elapsed seconds since the beginning of the execution of *command*
     - L is the basename of log files
     - T is the type of standard stream typescripted (I for input, O for output, E for error)
     - D is the current date with format year-month-day hour:minute:second
     - P is the process id of the process `log4t`
     - N is the line number (one separate numbering for each standard stream)

### Return status:
   * If *command* is invoked, the exit status of log4t will be the exit status of *command*.
   * Otherwise, `log4t` will exit with one of the following values:

      - 1-125    an error occurred in `log4t`.
      - 126      *command* was found but could not be invoked.
      - 127      *command* is unspecified, empty or could not be found.

### Examples:
   `log4t ls` invokes the executable /bin/ls
      standard and error output written to files named (for example) `ls.20210612211653.6518.log` and `ls.20210612211653.6518.err`.

   `log4t +O -l script.ksh >/dev/null`
      invokes the executable script `script.ksh`,
      standard output is not sent to the terminal (> /dev/null) but is logged anyway (+O) with a long format (-l).

   `log4t -n ksh -c "pwd ; ls -1 | wc -l"` :
      the command /usr/bin/ksh is invoked and executed without creating a log file (-n).

   `log4t -r -o output ksh -c 'sleep 1 ; print 1 ; sleep 1 ; print -u2 2 ; sleep 1 ; return 7' ; print $?` :
      log4t return code is the return code (7) of the command launched, ksh in this example.

   `log4t -Io test ksh -c 'i=0 ; while (( i=i+1 )) ; do sleep 10 ; print $i ; done' &` : 
      the user can logout without killing the process launched in the background (&) (note the needed option -I used since).

   `log4t $SHELL`
      makes a time-stamped record of a terminal session.

   `log4t -ro mylog +O cat log4t | wc -c ; cat log4t | wc -c` :
      the command "cat log4t" is executed and its standard output is logged (+O) to file mylog.log (-ro wc) while piped unmodified to command wc -l.

   `log4t` can also be used to log scheduled tasks, such as with cron, adding the following to the crontab:
      `28 14 * * * logit +OE ls /home/laurent aaaaa >/dev/null`
      
   the command `ls /home/laurent aaaaa >/dev/null` is executed and logged. Errors are reported by the system ans logged in the error log file.
