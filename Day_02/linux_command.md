 # Linux
 > Linux is a popular open-source operating system perfect for large server solutions. Almost all of the big tech companies have their servers running on Linux. 


 1. File System Navigation:
pwd: Print the current working directory.

ls: List files and directories in the current directory.

ls -l: Display detailed information about files and directories.

ls -a: Include hidden files and directories in the list.

cd directory: Change the current directory to the specified directory.

cd ..: Go up one level in the directory hierarchy.

cd ~: Go to the home directory.

mkdir directory: Create a new directory with the specified name.

rm file: Remove a file.

rm -r directory: Remove a directory and its contents recursively.

cp source destination: Copy a file or directory.

cp -r source destination: Copy a directory and its contents recursively.

mv source destination: Move or rename a file or directory.


**Move `text.py` into `FullStack` folder**

If `FullStack` doesn't exist yet — create it first:
```
mkdir FullStack
```
Then move the file:
```
mv text.py FullStack/
```

If `FullStack` exists one level up:
```
mv text.py ../FullStack/
```

If `FullStack` exists at a specific path:
```
mv text.py /full/path/to/FullStack/
```


2. File Manipulation:
cat file: Display the contents of a file.

less file: View the contents of a file with pagination.

head file: Display the first few lines of a file.

tail file: Display the last few lines of a file.

grep pattern file: Search for a pattern in a file.

grep -r pattern directory: Recursively search for a pattern in files within a directory.


3. File Permissions:
chmod permissions file: Change the permissions of a file or directory.

chmod +x file: Add executable permission to a file.

chmod -w file: Remove write permission from a file.

chown user:group file: Change the owner and group of a file or directory.

chgrp group file: Change the group of a file or directory.


4. Process Management:
ps: Display information about active processes.

ps -ef: Display detailed information about all processes.

kill process_id: Terminate a process with the specified process ID.

killall process_name: Terminate all processes with the specified name.

top: Display real-time information about system processes.

bg: Put a process in the background.

fg: Bring a background process to the foreground.


5. System Information:
uname: Print system information.

uname -a: Print detailed system information.

whoami: Display the current user.

df: Show disk space usage.

free: Display memory usage.

uptime: Show system uptime.

ifconfig: Display network interface information.

ping host: Send ICMP echo requests to a host for network connectivity testing.


6. Text Editing:
nano file: Open a file for editing with the Nano editor.

vi file: Open a file for editing with the Vi editor.

grep pattern file: Search for a pattern in a file.

grep -r pattern directory: Recursively search for a pattern in files within a directory.

sed 's/old/new/g' file: Replace occurrences of a pattern with a new value in a file


7. Compression and Archiving:
tar cf archive.tar files: Create a tar archive of specified files.

tar xf archive.tar: Extract files from a tar archive.

gzip file: Compress a file using gzip compression.

gzip -d file.gz: Decompress a file compressed with gzip.


8. Networking:
ssh user@host: Connect to a remote host using SSH.

scp source destination: Securely copy files between local and remote hosts.

wget url: Download a file from a specified URL.

curl url: Transfer data to or from a server using various protocols.


9. User Management:
useradd username: Create a new user account.

userdel username: Delete a user account.

passwd username: Set or change the password for a user account.

su username: Switch to another user account.

sudo command: Execute a command with superuser (root) privileges.


10. Package Management:
apt-get install package: Install a package using the Advanced Package Tool (APT) package manager.

apt-get update: Update the local package index to retrieve the latest package information.

apt-get upgrade: Upgrade all installed packages to their latest versions.

apt-get remove package: Uninstall a package.

apt-cache search keyword: Search for packages based on a keyword.


11. File Transfer:
scp source destination: Securely copy files between local and remote hosts using the SSH protocol.

rsync source destination: Synchronize files and directories between local and remote hosts efficiently.

sftp user@host: Start an interactive secure file transfer session with a remote host.


12. System Monitoring and Maintenance:
top: Monitor system processes and resource usage in real-time.

htop: Interactive process viewer and system monitor.

du -sh directory: Display the total size of a directory.

df -h: Show disk space usage of mounted file systems.

reboot: Reboot the system.

shutdown: Shutdown the system.


13. Networking and Connectivity:
ifconfig: Configure and display network interface information.

ping host: Send ICMP echo requests to a host for network connectivity testing.

netstat: Display network connection information, routing tables, and interface statistics.

ssh user@host: Connect to a remote host using SSH.

nc host port: Open a TCP or UDP connection to a remote host on a specified port.


14. System Information:
uname -a: Display detailed system information including the kernel version.

lsb_release -a: Show distribution-specific information about the Linux distribution.

hostname: Print or set the system's hostname.

date: Display the current date and time.


15. File Permissions and Ownership:
chmod permissions file: Change the permissions of a file or directory.

chown user:group file: Change the owner and group of a file or directory.

chgrp group file: Change the group of a file or directory.

ls -l: List files and directories with detailed permission and ownership information.