# Metasploitable-SSH-Remote-Access-Lab
A beginner-friendly cybersecurity lab demonstrating authorized SSH remote access from Parrot OS to a Metasploitable 2 virtual machine. The project covers network configuration, IP identification, SSH connectivity, authentication, legacy SSH compatibility, remote Linux administration, troubleshooting, and SOC-focused security concepts.

A beginner-friendly cybersecurity lab demonstrating authorized remote access to a Metasploitable 2 virtual machine using SSH from Parrot OS.

> **⚠️ Disclaimer:** This project is performed only on personally owned/authorized virtual machines in an isolated lab environment. Metasploitable 2 is intentionally vulnerable and should never be exposed directly to the public Internet or an untrusted network.

---

## 📌 Project Overview

In this lab, I configured two virtual machines:

* **Parrot Security OS** — Administration / security testing machine
* **Metasploitable 2** — Intentionally vulnerable target machine

The main objective is to learn:

* Basic IP networking
* SSH remote access
* Linux command-line administration
* SSH authentication
* Legacy SSH compatibility
* Remote system shutdown
* Basic security observations
* How a SOC analyst can investigate remote login activity

### Lab Architecture

```text
                    Isolated Virtual Network
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
      ┌───────────────┐             ┌─────────────────┐
      │   Parrot OS   │             │  Metasploitable │
      │                │    SSH      │       2         │
      │ Admin/Security │ ──────────► │     Target      │
      │     Machine    │             │                 │
      └───────────────┘             └─────────────────┘
       192.168.56.x                  192.168.56.101
```

---

# 1. Lab Requirements

## Software

* Oracle VirtualBox
* Parrot Security OS
* Metasploitable 2

## Recommended Network

Use one of the following:

* Host-only Adapter
* Internal Network

Do **not** expose Metasploitable 2 directly to the Internet.

---

# 2. Configure the Virtual Machines

Both machines must be connected to the same virtual network.

### Parrot OS

VirtualBox:

```text
Settings
   ↓
Network
   ↓
Adapter 1
   ↓
Host-only Adapter
```

### Metasploitable 2

Use the same network:

```text
Settings
   ↓
Network
   ↓
Adapter 1
   ↓
Host-only Adapter
```

The exact IP addresses may be different in your environment.

---

# 3. Find the Metasploitable IP Address

Start Metasploitable 2 and log in locally.

Run:

```bash
ifconfig
```

Look for an address similar to:

```text
inet addr:192.168.56.101
```

The important value is:

```text
192.168.56.101
```

This is the target's IP address in my lab.

> Your IP address may be different.

---

# 4. Find the Parrot OS IP Address

On Parrot OS, run:

```bash
ip addr
```

Look for an IP address on the same virtual network.

Example:

```text
192.168.56.102
```

The machines should be on the same network.

Example:

```text
Parrot OS          192.168.56.102
Metasploitable     192.168.56.101
```

---

# 5. Test Network Connectivity

From Parrot OS:

```bash
ping 192.168.56.101
```

A successful result may look like:

```text
64 bytes from 192.168.56.101: icmp_seq=1 ttl=64 time=0.5 ms
64 bytes from 192.168.56.101: icmp_seq=2 ttl=64 time=0.4 ms
```

Press:

```text
Ctrl + C
```

to stop the ping.

### What does this prove?

It shows that Parrot OS can communicate with the Metasploitable VM over the lab network.

---

# 6. Check Whether SSH Is Available

From Parrot OS:

```bash
nmap -p 22 192.168.56.101
```

Example:

```text
PORT   STATE SERVICE
22/tcp open  ssh
```

This means the SSH service is listening on TCP port 22.

---

# 7. First SSH Connection Attempt

The basic SSH syntax is:

```bash
ssh username@IP_ADDRESS
```

For example:

```bash
ssh msfadmin@192.168.56.101
```

However, modern OpenSSH clients may reject the old SSH algorithms used by Metasploitable 2.

You may see:

```text
Unable to negotiate with 192.168.56.101 port 22:
no matching host key type found.
Their offer: ssh-rsa,ssh-dss
```

---

# 8. Understanding the SSH Error

Metasploitable 2 is an old intentionally vulnerable operating system.

It offers legacy SSH algorithms such as:

```text
ssh-rsa
ssh-dss
```

Modern SSH clients disable some legacy algorithms by default.

Therefore, the connection can fail even though:

* The IP address is correct
* Port 22 is open
* SSH is running

---

# 9. Connect Using SSH

For this isolated lab, allow the legacy RSA host-key algorithm only for this connection:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa msfadmin@192.168.56.101
```

### Breaking down the command

```text
ssh
```

Use the Secure Shell protocol.

```text
-o
```

Specify an SSH option.

```text
HostKeyAlgorithms=+ssh-rsa
```

Temporarily allow the legacy `ssh-rsa` host-key algorithm.

```text
msfadmin
```

Username on Metasploitable.

```text
@
```

Separates the username from the target.

```text
192.168.56.101
```

IP address of the Metasploitable VM.

---

# 10. Authenticate

When SSH asks:

```text
msfadmin@192.168.56.101's password:
```

enter the credentials for your Metasploitable installation.

For the standard Metasploitable 2 image, the default account is commonly:

```text
Username: msfadmin
Password: msfadmin
```

When entering a Linux password, characters normally do not appear on the screen.

Press:

```text
Enter
```

after typing the password.

---

# 11. Successful SSH Login

A successful login should give you a remote shell, for example:

```text
msfadmin@metasploitable:~$
```

This means:

```text
Parrot OS
     │
     │ SSH
     ▼
Metasploitable 2
     │
     ▼
Remote Linux shell
```

You are now executing commands on the Metasploitable machine rather than on Parrot.

---

# 12. Verify the Remote Machine

After logging in, run:

```bash
whoami
```

Expected:

```text
msfadmin
```

Check the hostname:

```bash
hostname
```

Check the IP configuration:

```bash
ifconfig
```

Check the current directory:

```bash
pwd
```

List files:

```bash
ls
```

These commands help confirm that you are working on the remote machine.

---

# 13. Check the Current User

Run:

```bash
whoami
```

This answers:

> Which user account am I currently using?

Example:

```text
msfadmin
```

---

# 14. Check the Logged-In Session

Run:

```bash
who
```

or:

```bash
w
```

These commands can show information about logged-in users and sessions.

This is useful from a SOC perspective because authentication and user-session information can help analysts investigate remote access.

---

# 15. Check SSH-Related Processes

Run:

```bash
ps aux | grep ssh
```

This can help identify SSH-related processes.

---

# 16. Check Network Connections

Run:

```bash
netstat -tulnp
```

If `netstat` is unavailable, try:

```bash
ss -tulnp
```

Look for services listening on network ports.

---

# 17. Basic Remote Administration

Once connected through SSH, you can perform normal authorized Linux administration.

Examples:

```bash
pwd
```

```bash
ls
```

```bash
hostname
```

```bash
date
```

```bash
whoami
```

```bash
df -h
```

```bash
free -m
```

---

# 18. Remote Shutdown

Because Metasploitable is your own lab VM, you can shut it down remotely.

Run:

```bash
sudo shutdown -h now
```

Alternatively:

```bash
sudo poweroff
```

You may be asked for the account password.

The shutdown process is:

```text
Parrot OS
    │
    │ SSH
    ▼
Metasploitable
    │
    │ sudo shutdown
    ▼
System powers off
```

---

# 19. Exit the SSH Session

To leave the remote machine:

```bash
exit
```

or press:

```text
Ctrl + D
```

You should return to your Parrot terminal.

Example:

```text
msfadmin@metasploitable:~$ exit
logout

cyber@parrot:~$
```

---

# 20. Security Concepts Learned

This lab demonstrates several important cybersecurity concepts.

### SSH

SSH provides remote command-line access to another computer.

Default SSH port:

```text
TCP/22
```

### IP Address

The IP address identifies the machine on the network.

Example:

```text
192.168.56.101
```

### Authentication

SSH requires authentication before allowing access.

In this lab:

```text
Username → msfadmin
Password → configured account password
```

### Host Key

SSH uses host keys to help establish the identity of the remote server.

### Legacy Algorithms

Old systems may use cryptographic algorithms that modern clients no longer accept by default.

This is why Metasploitable required:

```bash
-o HostKeyAlgorithms=+ssh-rsa
```

---

# 21. SOC Analyst Perspective

This lab can also be viewed from a defensive/SOC perspective.

Imagine an analyst receives an alert:

```text
Remote SSH Login Detected
```

The analyst would want to investigate:

```text
Source IP
Destination IP
Destination Port
Username
Timestamp
Authentication result
Number of attempts
```

Example:

```text
Source:      192.168.56.102
Destination: 192.168.56.101
Protocol:    SSH
Port:        22
User:        msfadmin
Result:      Successful
```

This is the type of information that can later be sent to a SIEM such as Wazuh or Splunk.

---

# 22. Lab Troubleshooting

## Error: No matching host key type

Example:

```text
no matching host key type found
```

Use:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa msfadmin@TARGET_IP
```

---

## Error: Connection refused

Check whether SSH is running on Metasploitable:

```bash
sudo service ssh status
```

Also check:

```bash
nmap -p 22 TARGET_IP
```

---

## Error: Connection timed out

Check:

1. Both VMs are running.
2. Both VMs use the same VirtualBox network.
3. The IP address is correct.
4. Test with:

```bash
ping TARGET_IP
```

---

## Error: Permission denied

Check the username and password.

For the standard Metasploitable 2 image:

```text
Username: msfadmin
Password: msfadmin
```

---

# 23. Security Warning

Metasploitable 2 is intentionally vulnerable.

Do not:

* Expose it to the public Internet.
* Use it against systems you do not own.
* Port-forward it from your router.
* Put it on an untrusted network.
* Use these techniques against unauthorized systems.

Keep the environment isolated:

```text
                YOUR COMPUTER
                     │
              VirtualBox Network
                     │
             ┌───────┴───────┐
             │               │
             ▼               ▼
          Parrot       Metasploitable
         Security          2
```

---

# 24. What I Learned

Through this project, I learned:

* How virtual machines communicate over a private network.
* How to identify a target VM's IP address.
* How to test network connectivity using `ping`.
* How to identify an open SSH port using Nmap.
* How SSH remote access works.
* How SSH usernames and IP addresses are structured.
* Why legacy SSH algorithms can cause compatibility problems.
* How to perform basic remote Linux administration.
* How to terminate an SSH session.
* How remote login activity can be relevant to SOC monitoring.

---

# 25. Future Improvements

I plan to extend this lab by adding:

* SSH authentication log analysis
* Failed SSH login investigation
* Successful SSH login investigation
* Wazuh monitoring
* SIEM log collection
* Brute-force detection in an isolated lab
* MITRE ATT&CK mapping
* SSH hardening
* Password authentication vs SSH keys
* Incident investigation based on SSH logs

---

# 26. Final Result

The completed lab demonstrates:

```text
                 SSH Remote Access Lab

                    Parrot OS
                192.168.56.x
                      │
                      │
                  TCP/22 SSH
                      │
                      ▼
              Metasploitable 2
                192.168.56.101
                      │
                      ▼
                 msfadmin
                Remote Shell
```

This project provides a foundation for learning **Linux administration, networking, SSH, authentication, and SOC monitoring** in a controlled cybersecurity environment.
