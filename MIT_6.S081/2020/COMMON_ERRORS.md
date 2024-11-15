# Common Errors

After installing all the required deps and you run `make qemu`, you will face this error:
```bash
user/sh.c: In function 'runcmd':
user/sh.c:60:1: error: infinite recursion detected [-Werror=infinite-recursion]
   60 | runcmd(struct cmd *cmd)
      | ^~~~~~
user/sh.c:91:5: note: recursive call
   91 |     runcmd(rcmd->cmd);
      |     ^~~~~~~~~~~~~~~~~
user/sh.c:111:7: note: recursive call
  111 |       runcmd(pcmd->left);
      |       ^~~~~~~~~~~~~~~~~~
user/sh.c:118:7: note: recursive call
  118 |       runcmd(pcmd->right);
      |       ^~~~~~~~~~~~~~~~~~~
user/sh.c:97:7: note: recursive call
   97 |       runcmd(lcmd->left);
      |       ^~~~~~~~~~~~~~~~~~
user/sh.c:99:5: note: recursive call
   99 |     runcmd(lcmd->right);
      |     ^~~~~~~~~~~~~~~~~~~
user/sh.c:129:7: note: recursive call
  129 |       runcmd(bcmd->cmd);
      |       ^~~~~~~~~~~~~~~~~
cc1: all warnings being treated as errors
make: *** [user/sh.o] Error 1
```

This is because in the `Makefile` `CFLAGS`, we have the `-Werror` flag (line 70) which tells the compiler (gcc/clang) to treat all warnings as errors.
This means any warnigs that would normally just display a message will instead cause the compilation to fail.

## Fix

1. Go to the file `user/sh.c` and find the function `runcmd(struct cmd *cmd)`

2. Make the following file changes to it:

```C
// Execute cmd.  Never returns.
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Winfinite-recursion"
void
runcmd(struct cmd *cmd)
{
  int p[2];
  struct backcmd *bcmd;
  struct execcmd *ecmd;
  struct listcmd *lcmd;
  struct pipecmd *pcmd;
  struct redircmd *rcmd;

  if(cmd == 0)
    exit(1);

  switch(cmd->type){
  default:
    panic("runcmd");

  case EXEC:
    ecmd = (struct execcmd*)cmd;
    if(ecmd->argv[0] == 0)
      exit(1);
    exec(ecmd->argv[0], ecmd->argv);
    fprintf(2, "exec %s failed\n", ecmd->argv[0]);
    break;

  case REDIR:
    rcmd = (struct redircmd*)cmd;
    close(rcmd->fd);
    if(open(rcmd->file, rcmd->mode) < 0){
      fprintf(2, "open %s failed\n", rcmd->file);
      exit(1);
    }
    runcmd(rcmd->cmd);
    break;

  case LIST:
    lcmd = (struct listcmd*)cmd;
    if(fork1() == 0)
      runcmd(lcmd->left);
    wait(0);
    runcmd(lcmd->right);
    break;

  case PIPE:
    pcmd = (struct pipecmd*)cmd;
    if(pipe(p) < 0)
      panic("pipe");
    if(fork1() == 0){
      close(1);
      dup(p[1]);
      close(p[0]);
      close(p[1]);
      runcmd(pcmd->left);
    }
    if(fork1() == 0){
      close(0);
      dup(p[0]);
      close(p[0]);
      close(p[1]);
      runcmd(pcmd->right);
    }
    close(p[0]);
    close(p[1]);
    wait(0);
    wait(0);
    break;

  case BACK:
    bcmd = (struct backcmd*)cmd;
    if(fork1() == 0)
      runcmd(bcmd->cmd);
    break;
  }
  exit(0);
}
#pragma GCC diagnostic pop
```

The key changes are:
  1. Added `#pragma GCC diagnostic push` to save the current warning settings
  2. Added `#pragma GCC diagnostic ignored "-Winfinite-recursion"` to disable the specific warning
  3. Added `#pragma GCC diagnostic pop` to restore the warning settings after the function

This is safe because:

  - Each recursive call is always in a different process (after fork)
  - The recursion will terminate when we hit an EXEC command
  - The shell command structure is inherently recursive (commands can contain other commands)
  - Each path either:

      - Calls `exec()` which replaces the current process
      - Or exits explicitly
      - Or makes a recursive call in a child process

The pragmas tell the compiler that we understand there's recursion here and it's intentional. This resolves the compilation error while maintaining the correct shell behavior.

## Additional Resources

- [GCC Diagnostic Pragmas Documentation](https://gcc.gnu.org/onlinedocs/gcc/Diagnostic-Pragmas.html)
- [GCC Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
