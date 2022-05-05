# PrivEsc via \*rc, profile and config files

## Theory

### Bash Startup Files

This section describes how Bash executes its startup files. If any of the files exist but cannot be read, Bash reports an error.

#### **Invoked as an interactive login shell, or with --login**

When Bash is invoked as an interactive login shell, or as a non-interactive shell with the `--login` option, it first reads and executes commands from the file /etc/profile, if that file exists. After reading that file, it looks for `~/.bash_profile`, `~/.bash_login`, and `~/.profile`, in that order, and reads and executes commands from the first one that exists and is readable. The `--noprofile` option may be used when the shell is started to inhibit this behavior.

When an interactive login shell exits, or a non-interactive login shell executes the `exit` builtin command, Bash reads and executes commands from the file `~/.bash_logout`, if it exists.

#### **Invoked as an interactive non-login shell**

When an interactive shell that is not a login shell is started, Bash reads and executes commands from `~/.bashrc`, if that file exists. This may be inhibited by using the `--norc` option (this option may be useful since it may be the case that the .bashrc is executing a command to close the interactive shell after a few minutes.). The `--rcfile` file option will force Bash to read and execute commands from file instead of \~/.bashrc.

So, typically, your `~/.bash_profile` contains the line:

```
if [ -f ~/.bashrc ]; then . ~/.bashrc; fi
```

after (or before) any login-specific initializations.

### .config directory

**.config** is a convention, defined by [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

* There is a single base directory (`.config`) relative to which user-specific configuration files should be written. This directory is defined by the environment variable `$XDG_CONFIG_HOME`.

Then, the configuration files for some programs are located in the \~/.config directory (depending on the **$XDG\_CONFIG\_HOME** variable), in the following format:

```
~/.config/<program-name>/*.conf
```

## Practice

### .bashrc/.bash\_profile (Privilege Escalation)

If we can write content in the `.bashrc` or `.bash_profile` files of other users, we can insert a malicious payload in some of the lines.

### **XDG\_CONFIG\_HOME** variable (Privilege Escalation)

* In this case, we need 2 things: The variable **$XDG\_CONFIG\_HOME** must be modifiable:

```bash
# Look at the env_keep part:
bob@box~$ sudo -l

env_reset, secure_path=..., env_keep+=XDG_CONFIG_HOME
```

* A program that runs as another user and its configuration file (which must be found in our home directory) must have some way to execute commands or perform a malicious action. If you see that it is not formatted, perhaps simply adding a command will work.

### Others \*rc files

There may be cases where we can run an application as root that needs to load an .rc file. However, if the file does not exist in root-owned directories and looks for it in our home directory (/home//.rc), we can create our own and try to take advantage of some variable, configuration or even command injection. A case of this can be the [AXEL](https://github.com/axel-download-accelerator/axel) application (with the possibility of executing root and non-existent .\*rc files).
