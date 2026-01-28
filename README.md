# Wargame Writeups: OverTheWire Bandit

**Focus:** Linux SysAdmin, Bash Scripting, SSL/TLS, Networking
**Tools Used:** Bash, OpenSSL, Nmap, Netcat, SSH, Gzip/Tar

## Overview
This repository documents my solutions and technical analysis for the **OverTheWire: Bandit** wargame. My goal is not just to capture the flag, but to document the **underlying systems engineering concepts** (file permissions, SSL handshakes, SUID binaries) required to solve each level.

---

## Level Walkthroughs (0 → 20)

### Levels 0 → 5: Navigation & File Handling

* **Level 0 → 1:**
    * **Objective:** Access the server and read a file named `readme`.
    * **Command:** `ls -al` then `cat readme`
    * **Why:** Used `ls -al` to list all files (including hidden ones) to verify the directory contents.

* **Level 1 → 2:**
    * **Objective:** Read a file named `-` (dash).
    * **Command:** `cat ./-`
    * **Why:** The dash is usually interpreted as "standard input." Specifying the path `./-` forces the shell to treat it as a file.

* **Level 2 → 3:**
    * **Objective:** Read a file with spaces in the name.
    * **Command:** `cat ./"spaces in this filename"`
    * **Why:** Quotes ensure the shell treats the entire string as a single filename rather than separate arguments.

* **Level 3 → 4:**
    * **Objective:** Access a hidden file inside the `inhere` directory.
    * **Command:** `ls -al` then `cat .Hiding-From-You`
    * **Why:** Standard `ls` does not show files starting with a dot. Added `-al` to reveal hidden files.

* **Level 4 → 5:**
    * **Objective:** Find the only human-readable file in the `inhere` directory.
    * **Command:** `file ./*` then `cat ./-file07`
    * **Why:** Used the `file` command to determine data types, isolating the ASCII text file among binary data.

* **Level 5 → 6:**
    * **Objective:** Find a file: 1033 bytes, human-readable, not executable.
    * **Command:** `find . -size 1033c`
    * **Why:** Used `find` with the `-size` flag to filter by byte count, instantly locating the correct file in nested directories.

### Levels 6 → 10: Find, Grep, and Permissions

* **Level 6 → 7:**
    * **Objective:** Find a file owned by user `bandit7`, group `bandit6`, sized 33 bytes.
    * **Command:** `find / -size 33c -user bandit7 -group bandit6 2>/dev/null`
    * **Why:** Redirected "Permission Denied" errors to `/dev/null` to clean the output and find the hidden password file.

* **Level 7 → 8:**
    * **Objective:** Find the password stored next to the word "millionth".
    * **Command:** `grep "millionth" data.txt`
    * **Why:** Used `grep` to filter a large dataset for a specific string pattern.

* **Level 8 → 9:**
    * **Objective:** Find the only line of text that occurs exactly once.
    * **Command:** `sort data.txt | uniq -u`
    * **Why:** `uniq` requires sorted input to detect duplicates. Piped `sort` into `uniq -u` to isolate the unique line.

* **Level 9 → 10:**
    * **Objective:** Find a human-readable string in a binary file.
    * **Command:** `strings data.txt | grep "===="`
    * **Why:** Used `strings` (or `tr`) to strip non-printable characters from the binary file.

* **Level 10 → 11:**
    * **Objective:** Decode a Base64 encoded file.
    * **Command:** `base64 --decode data.txt`
    * **Why:** Recognized standard Base64 encoding and used the native decode tool.

### Levels 11 → 15: Encoding & Networking

* **Level 11 → 12:**
    * **Objective:** Decode a ROT13 shifted string.
    * **Command:** `cat data.txt | tr 'a-zA-Z' 'n-za-mN-ZA-M'`
    * **Why:** ROT13 rotates characters by 13 places. Used `tr` to manually map the alphabet shift.

* **Level 12 → 13:**
    * **Objective:** Extract a file repeatedly compressed (hexdump, tar, gzip, bzip2).
    * **Methodology:** Used `xxd -r` to reverse the hexdump, then `file` to identify compression types (gzip/bzip2), repeatedly decompressing until the text was revealed.

* **Level 13 → 14:**
    * **Objective:** Use a provided SSH private key to log in.
    * **Command:** `ssh -i sshkey.private bandit14@localhost`
    * **Why:** Used the `-i` flag to authenticate via key file instead of password.

* **Level 14 → 15:**
    * **Objective:** Submit the current password to port 30000 on localhost.
    * **Command:** `telnet localhost 30000`
    * **Why:** Connected directly to the local network daemon to retrieve the next flag.

* **Level 15 → 16:**
    * **Objective:** Submit password to an SSL-encrypted service.
    * **Command:** `openssl s_client -connect localhost:30001`
    * **Why:** Telnet does not support SSL. Used `openssl` to handle the secure handshake.

### Levels 16 → 20: Port Scanning & SSH Tricks

### Level 16 → 17: The OpenSSL "Quiet" Trick
* **Objective:** Scan ports for an SSL server and retrieve the credentials.
* **Analysis:**
  * I found the server on port 31518 using `nmap`.
  * However, `openssl s_client` defaults to **interactive mode**. This means every key pressed is interpreted as a command rather than data.
  * According to the manpage, `R` renegotiates the session and `Q` ends it. Since the password might contain these characters, sending it raw causes the connection to hang or close.
* **Command:** `openssl s_client -connect localhost:31518 -quiet`
* **Why:** The `-quiet` flag prevents the interactive behavior and lets me send data uninhibited.

**Resulting RSA Key:**
```text
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZ.zeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMIOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZ187ORIO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
... (Key abbreviated for readability, full key stored locally) ...
dxviW8+TFVEBI1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----
```

* **Level 17 → 18:**
    * **Objective:** Find the difference between two files.
    * **Command:** `diff passwords.old passwords.new`
    * **Why:** Used `diff` to instantly highlight the changed line containing the new password.

* **Level 18 → 19:**
    * **Objective:** Execute a command on a server that forces logout.
    * **Command:** `ssh bandit18@... "cat readme"`
    * **Why:** Appended the command to the SSH request to execute it *before* the remote shell's `.bashrc` could force a logout.
