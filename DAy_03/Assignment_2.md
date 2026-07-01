# Linux Lab Assignment – 1
**Topics:** User Management, Process Management, Package Installation, Networking Commands

---

## Part A – User Management

**1. Create a user named `student1`**
```bash
sudo useradd student1
or 
sudo useradd -m student1

-m = create the home directory automatically
```

**2. Set a password for the user**
```bash
sudo passwd student1
```

**3. Create a group named `developers`**
```bash
sudo groupadd developers
```

**4. Add `student1` to the `developers` group**
```bash
sudo usermod -aG developers student1
```

**5. Verify the groups of `student1`**
```bash
groups student1

O/P:
student1 : student1 developers
```
> student1 is in both their own default group (student1) and the developers group you added them to.

**6. Switch to the `student1` account**
```bash
su - student1
```
> - (dash) (not attached to a letter) means "start a fresh login shell"

**7. Return to the original user**
```bash
exit
```

---

## Part B – Process Management

**1. Display all running processes**
```bash
ps aux
```

**2. Find the PID of the `sshd` or `bash` process**
```bash
pgrep sshd
pgrep bash
```
>pgrep bash
1308
1736
1745
You got three PIDs back — that means there are three bash sessions currently running on this machine (different terminals/shells open).
>pgrep sshd
1221
That's the PID of the sshd daemon — the process quietly listening in the background, ready to accept any incoming SSH connections to this machine.


**3. Start a background process using `sleep` for 300 seconds**
```bash
sleep 300 &
```

**4. Display the running background process**
```bash
jobs
```

**5. Stop (kill) the background process**
```bash
kill %1
```

**6. Display system uptime**
```bash
uptime
```

---

## Part C – Package Management

**1. Update the package repository**
```bash
sudo apt update
```

**2. Install the `tree` package**
```bash
sudo apt install tree -y
```

**3. Verify that the package is installed**
```bash
dpkg -l | grep tree
```

**4. Display the version of the installed package**
```bash
tree --version
```

**5. Remove the package**
```bash
sudo apt remove tree -y
```

---

## Part D – Networking Commands

**1. Display your IP address**
```bash
ip addr
```

**2. Display your hostname**
```bash
hostname
```

**3. Check connectivity to google.com**
```bash
ping -c 4 google.com
```

**4. Display all listening ports**
```bash
ss -tuln
```

**5. Display the routing table**
```bash
ip route
```

**6. Find the IP address of www.amazon.com**
```bash
nslookup www.amazon.com
```

---

## Bonus Task – Install & Verify Nginx

**Install Nginx**
```bash
sudo apt install nginx -y
```

**Verify the service is running**
```bash
sudo systemctl status nginx
```

---

## Notes
- Run `sudo apt update` once at the start so package commands work smoothly.
- Take a screenshot after each command's output for submission.
- If `sudo` gives `command not found`, you're likely already root — drop `sudo` and run the command directly.