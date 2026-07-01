root@ubuntu:~$ whoiam
whoiam: command not found
root@ubuntu:~$ whoami
root
root@ubuntu:~$ user id
Command 'user' not found, did you mean:
  command 'fuser' from deb psmisc (23.6-1)
  command 'users' from deb coreutils (9.4-3ubuntu6.2)
  command 'userv' from deb userv (1.2.1~beta4)
  command 'iuser' from deb ipmiutil (3.1.9-3)
Try: apt install <deb name>
root@ubuntu:~$ id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:~$ id username
id: 'username': no such user
root@ubuntu:~$ id root
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:~$ adduser Arpit
err: Please enter a username matching the regular expression
            configured via the NAME_REGEX configuration variable.  Use the
            `--allow-bad-names' option to relax this check or reconfigure
            NAME_REGEX in configuration.
root@ubuntu:~$ adduser arpit
info: Adding user `arpit' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `arpit' (1001) ...
info: Adding new user `arpit' (1001) with group `arpit (1001)' ...
info: Creating home directory `/home/arpit' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for arpit
Enter the new value, or press ENTER for the default
        Full Name []: Arpit Verma
        Room Number []: 506
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] y
info: Adding new user `arpit' to supplemental / extra groups `users' ...
info: Adding user `arpit' to group `users' ...
root@ubuntu:~$ useradd -m -s /bin/bash arpit
useradd: user 'arpit' already exists
root@ubuntu:~$ adduser anubhav
info: Adding user `anubhav' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `anubhav' (1002) ...
info: Adding new user `anubhav' (1002) with group `anubhav (1002)' ...
info: Creating home directory `/home/anubhav' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for anubhav
Enter the new value, or press ENTER for the default
        Full Name []: Anubhav Raja
        Room Number []: 506
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] y
info: Adding new user `anubhav' to supplemental / extra groups `users' ...
info: Adding user `anubhav' to group `users' ...
root@ubuntu:~$ passwd arpit 
New password: 
Retype new password: 
Sorry, passwords do not match.
passwd: Authentication token manipulation error
passwd: password unchanged
root@ubuntu:~$ paswd anubhav
Command 'paswd' not found, did you mean:
  command 'passwd' from deb passwd (1:4.13+dfsg1-4ubuntu3.2)
Try: apt install <deb name>
root@ubuntu:~$ passwd anubhav
New password: 
Retype new password: 
passwd: password updated successfully
root@ubuntu:~$ usermod  -aG sudo anubhav
root@ubuntu:~$ adduser ankit
info: Adding user `ankit' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `ankit' (1003) ...
info: Adding new user `ankit' (1003) with group `ankit (1003)' ...
info: Creating home directory `/home/ankit' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
Sorry, passwords do not match.
passwd: Authentication token manipulation error
passwd: password unchanged
Try again? [y/N] y
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for ankit
Enter the new value, or press ENTER for the default
        Full Name []: Ankit Roy
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] 
info: Adding new user `ankit' to supplemental / extra groups `users' ...
info: Adding user `ankit' to group `users' ...
root@ubuntu:~$ 
root@ubuntu:~$ userdel ankit
root@ubuntu:~$ groupadd developers
root@ubuntu:~$ su - anubhav
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

anubhav@ubuntu:~$ su anubhav
Password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

anubhav@ubuntu:~$ su anubhav
Password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

anubhav@ubuntu:~$ sudo su anubhav
[sudo] password for anubhav: 
anubhav@ubuntu:~$ 


# sudo
> Superuser Do

> gives you temporary admin permission (needed because creating users is a system-level change)
>sudo is only needed when a command modifies something (creating users, installing packages, editing system files). 
>Reading data never needs admin permission like... groups student1
>remember the rule: read-only/harmless commands don't need sudo
>No sudo needed if you're already root — but if you're a regular user, you'd use sudo su - student1

# useradd
> Creates a new user account on the system.

# passwd
>  Sets or changes a password for a user account.

# groupadd
> Creates a new group — groups are used to bundle permissions for multiple users at once (instead of setting permissions per user).

# usermod
> user modify
> Modifies an existing user's settings — like adding them to a group, changing their home directory, shell, etc

# -aG
>append Group
>-a = append (don't remove student1 from any groups they're already in)
>-G = specify the secondary group(s) to add

>Without -a, you'd risk wiping out all their other group memberships — important habit to remember!

>Structure to remember:
>sudo   [command]   [flags]   [arguments]
>sudo   usermod     -aG       developers student1

# Command

>The actual program/tool you're running
Tells Linux what action to perform
Example: usermod, ls, ping, apt

# Flag (also called an "option")

>Modifies how the command behaves
Usually starts with - (short form) or -- (long form)
Example: -aG in usermod -aG developers student1

-a = append
-G = target a group


You can often combine short flags: -aG is really -a + -G stuck together

# Argument

>The actual thing the command acts on
No dash in front — just plain text/values
Example: developers and student1 are arguments — they tell usermod which group and which user to modify

# su ("switch user")
>- (dash) = also load student1's environment/home directory (like a fresh login), not just their permissions

>single dash - (short flag)

Used for short, one-letter options
Example: -a, -G, -c
Can be combined: -aG = -a + -G

>Double dash -- (long flag)

Used for full-word options — more readable, less cryptic
Example: --append, --group, --help
Can't be combined like short flags
Often the same option has both a short and long version:

-a and --append do the same thing
-h and --help do the same thing

# ps aux
>ps

>Full form: Process Status
>Lists currently running processes on the system

>Flags aux (three combined):

a = show processes for all users (not just yours)
u = display in user-friendly format (shows user, %CPU, %MEM, etc.)
x = include processes without a controlling terminal (background/system processes)

USER   PID  %CPU  %MEM   VSZ   RSS TTY   STAT START   TIME COMMAND
root     1   0.0   0.1  1234   568 ?     Ss   10:00   0:01 /sbin/init
root   842   0.0   0.3  9876  3200 ?     S    10:01   0:00 /usr/sbin/sshd
student1 1023 0.1  0.5 12000  5400 pts/0 R+   10:15   0:00 bash


USER--Who owns/started this process
PID --Process ID — unique number identifying this process (you'll need this to kill it later)
%CPU--How much CPU this process is currently using
%MEM--How much RAM this process is using
VSZ--Virtual memory size (total memory the process could use)RSS -Resident memory (actual physical RAM being used right now)
TTY--Which terminal it's attached to — ? means no terminal (background/system process)
STAT--Process state — S = sleeping, R = running, Z = zombie (dead but not cleaned up), + = foreground
START--When the process started
TIME--Total CPU time consumed since it started
COMMAND--The actual command/program that launched 

# SSH
>Full form: Secure SHell
>It's a protocol (a set of rules) for securely connecting to another computer over a network
>Everything sent through SSH is encrypted — so even on public networks, nobody can read your commands, passwords, or data in transit

# sshd
>Full form: SSH Daemon
>A "daemon" = a background program that's always running, waiting to do a job
>sshd specifically waits for someone to connect remotely (like when you SSH into an EC2 instance) and handles that login
>It's basically the "doorman" that lets you SSH into a server

# bash

>Full form: Bourne Again SHell
>This is the actual program running your terminal right now
>Every time you type a command, bash is the thing reading it and executing it
So right now, as you're typing in your terminal — you're inside a running bash process

# pgrep

>Full form: Process grep (grep(Global Regular Expression Print) = search tool for finding text patterns)
>grep searches inside files for a word
>pgrep instead searches through the list of running processes for one matching a name, and gives you back its PID
>It's basically a shortcut so you don't have to run ps aux and scroll through everything yourself

# pgrep bash

>Searches for all processes whose name is bash
Found 3 matches: 1308, 1736, 1745
>Why 3? Because multiple bash shells are open right now — maybe different terminal tabs, or one bash spawned another. Each one gets its own unique PID.

# pgrep sshd

>Searches for all processes whose name is sshd
Found 1 match: 1221
>Why just 1? Because only one SSH daemon needs to run on a system — it's a single background service always listening for connections. There's no need for multiple copies of it.

# echo $$ breakdown:

>echo = print text to the screen
>$$ = a special shell variable that always holds the PID of the current shell