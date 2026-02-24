
There are multiple ways to upgrade a shell, but here I'll look into the `script / stty` trick:

```zsh
user@ssh-machine:~$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
user@ssh-machine:~$ ^Z
[1]+  Stopped                 nc -lnvp 443
my-user@my-machine$ stty raw -echo; fg
nc -lnvp 443
            reset
reset: unknown terminal type unknown
Terminal type? screen
user@ssh-machine:~$
```

The terminal will now work with most of the shortcuts we are used to