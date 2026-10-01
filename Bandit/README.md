# OTW BANDIT NOTES 

*Progress: Levels 0-24 complete (Level 25 in progress)*

---

## LEVEL 0 — SSH Basics

**GOAL:** Connect to the server via SSH

**CMD:**
```bash
  ssh bandit0@bandit.labs.overthewire.org -p 2220
  cat readme
```

**KEY CONCEPT:**

- SSH = Secure Shell. Used to remotely connect to servers.
- -p flag specifies non-default port (2220 instead of 22).

## LEVEL 1 — Reading files named "-"

**GOAL:** Read a file named "-"

**CMD:**
```bash
  cat ./-
  OR
  cat < -
```

**KEY CONCEPT:**

- Shell treats "-" as a flag, not a filename.
- "./" prefix tells shell it's a file path, not a flag.
- "<" redirect also bypasses this issue.

## LEVEL 2 — Files with spaces in name

**GOAL:** Read a file named "spaces in this filename"

**CMD:**
```bash
  cat "spaces in this filename"
  OR
  cat < 'spaces in this filename'
```

**KEY CONCEPT:**

- Spaces break shell arguments. Use quotes or backslash escape.
- Tab autocomplete also handles spaces automatically.

## LEVEL 3 — Hidden files

**GOAL:** Find a hidden file in a directory

**CMD:**
```bash
  cd inhere
  ls -a
  cat .<filename>
```

**KEY CONCEPT:**

- Files starting with "." are hidden in Linux.
- "ls" doesn't show them. "ls -a" shows ALL files including hidden.

## LEVEL 4 — Human-readable file among binaries

**GOAL:** Find the only human-readable file in inhere/

**CMD:**
```bash
  file ./*
```

**KEY CONCEPT:**

- "file" command tells you what type a file is (ASCII, binary, etc.)
- Look for "ASCII text" in the output — that's your readable file.
- "./*" = all files in current directory.

## LEVEL 5 — Find by properties

**GOAL:** Find file that is human-readable, 1033 bytes, not executable

**CMD:**
```bash
  find . -type f -size 1033c ! -executable
```

**KEY CONCEPT:**

- "find" is the most powerful search tool in Linux.
- -type f       → files only (not directories) -size 1033c   → exactly 1033 bytes (c = bytes) ! -executable → NOT executable Chain multiple conditions in one command.

## LEVEL 6 — Find file owned by specific user/group

**GOAL:** Find file owned by bandit7, group bandit6, size 33 bytes

**CMD:**
```bash
  find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

**KEY CONCEPT:**

- -user  → filter by owner username -group → filter by group name Search from "/" = entire server, not just current dir.
- "2>/dev/null" → hides "Permission denied" errors (redirects stderr to trash). Use this constantly in CyS work.

## LEVEL 7 — grep (search within file)

**GOAL:** Find password next to the word "millionth" in data.txt

**CMD:**
```bash
  grep "millionth" data.txt
```

**KEY CONCEPT:**

- grep searches for a pattern inside a file.
- Prints the entire line containing the match.
- Essential for log analysis and CTF work.

## LEVEL 8 — Unique line in file

**GOAL:** Find the line that appears only once in data.txt

**CMD:**
```bash
  sort data.txt | uniq -u
```

**KEY CONCEPT:**

- sort  → sorts lines alphabetically (groups duplicates together) uniq  → works on sorted data, removes/finds duplicates -u    → print ONLY lines that appear exactly once Pipe "|" sends output of one command as input to next.

## LEVEL 9 — Human-readable strings in binary file

**GOAL:** Find password preceded by "===" in binary file

**CMD:**
```bash
  strings data.txt | grep "==="
```

**KEY CONCEPT:**

- strings → extracts all human-readable text from any file Used in malware analysis and binary reverse engineering.
- Combine with grep to filter specific patterns.

## LEVEL 10 — Base64 decode

**GOAL:** Decode a base64 encoded file

**CMD:**
```bash
  base64 -d data.txt
```

**KEY CONCEPT:**

- base64 is an encoding scheme (not encryption).
- Strings ending with "=" or "==" are often base64.
- -d flag = decode. Without it = encode.
- Common in CTFs, malware payloads, and web tokens (JWT).

## LEVEL 11 — ROT13 cipher

**GOAL:** Decode ROT13 encoded file (letters rotated by 13 positions)

**CMD:**
```bash
  cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**KEY CONCEPT:**

- tr = translate/replace characters ROT13 = each letter shifted 13 places in alphabet 'A-Za-z' → 'N-ZA-Mn-za-m' maps each letter to its ROT13 pair Simple substitution cipher — an obfuscation technique, not real cryptography.

## LEVEL 12 — Hexdump + repeated compression layers

**GOAL:** Reverse a hexdump, then decompress multiple layers (gzip/bzip2/tar mixed, unknown order) until plain ASCII text with password appears.

**CMD:**
```bash
  mktemp -d
  cp ~/data.txt <tempdir>
  cd <tempdir>
  xxd -r data.txt > data
  file data                → tells you the NEXT compression type
  (repeat: rename with correct extension, decompress, file again)
  Note: tar xf does NOT delete the archive it extracted — after
  tar xf, always `ls -la` to find the NEW file it produced, don't
  re-check the old archive file.
```

**KEY CONCEPT:**

- Each layer only reveals itself after the previous one is stripped.
- `file` is the map — trust its output over assumption.
- tar extraction leaves the old archive on disk; forgetting this causes false "nothing happened" loops.
- Real-world use: malware droppers and forensic evidence often nest multiple compression/encoding layers specifically to slow down analysts doing exactly this kind of peeling.

## LEVEL 13 — SSH login using private key

**GOAL:** Use a private key file to SSH into bandit14 (bandit14's password file isn't readable from bandit13).

**CMD:**
```bash
  From LOCAL machine (not from inside a bandit session):
  ssh bandit13@bandit.labs.overthewire.org -p 2220 "cat sshkey.private" > ~/bandit14key.private
  chmod 600 ~/bandit14key.private
  ssh -i ~/bandit14key.private bandit14@bandit.labs.overthewire.org -p 2220
```

**KEY CONCEPT:**

- Manual copy-paste of a private key from a terminal is unreliable — terminal display-wrapping can turn real newlines into spaces, corrupting the base64 structure (symptom: "error in libcrypto").
- Fix: let SSH itself pull the file — run a single remote command via ssh and redirect its stdout straight to a local file. No human eyes/clipboard touch the content, so formatting survives.
- chmod 600 is mandatory — SSH refuses keys with open permissions.

## LEVEL 14 — Netcat (nc) — sending data to a port

**GOAL:** Submit bandit14's password to port 30000 on localhost.

**CMD:**
```bash
  cat /etc/bandit_pass/bandit14
  echo "<password>" | nc localhost 30000
```

**KEY CONCEPT:**

- Once logged in as bandit14, you can read your own password file directly (no permission issue now — you ARE that user).
- nc pipes data straight into a raw TCP connection; the service on port 30000 checks it and returns the next password.

## LEVEL 15 — SSL/TLS encrypted connection

**GOAL:** Submit password to port 30001 using SSL encryption

**CMD:**
```bash
  echo "PASSWORD" | openssl s_client -connect localhost:30001 -quiet
```

**KEY CONCEPT:**

- openssl s_client = connect to SSL/TLS encrypted services.
- Same as nc but wraps connection in encryption layer.
- "self-signed certificate" warning = normal in labs/CTFs.
- -quiet = suppress handshake noise, show only data.
- Real-world use: testing HTTPS services, SSL cert inspection.

## LEVEL 16 — Port scanning with service detection (SSL identification)

**GOAL:** Find the correct port in range 31000-32000, distinguish it from decoy echo servers, and retrieve a private key as reward.

**CMD:**
```bash
  nmap -sV localhost -p 31000-32000
  (identify SSL ports; echo services are decoys, "ssl/unknown" is
   the real target since its behavior doesn't match a known signature)
  echo "<bandit16_password>" | openssl s_client -connect localhost:<port> -quiet

  To safely capture the returned key without manual copy-paste:
  ssh -i ~/bandit14key.private bandit14@bandit.labs.overthewire.org -p 2220 \
    'echo "<password>" | openssl s_client -connect localhost:<port> -quiet' > ~/raw_output.txt

  sed -n '/-----BEGIN OPENSSH PRIVATE KEY-----/,/-----END OPENSSH PRIVATE KEY-----/p' ~/raw_output.txt > ~/bandit17.private
  chmod 600 ~/bandit17.private
  ssh -i ~/bandit17.private bandit17@bandit.labs.overthewire.org -p 2220
```

**KEY CONCEPT:**

- -sV flag = service/version detection, not just port state.
- Multiple open ports in a range can be decoys (echo servers) — distinguishing real target requires understanding WHAT each service should do when given correct input, not just scanning.
- sed range-print (/pattern1/,/pattern2/p) extracts an exact block from noisy output without touching a clipboard — same anti- corruption principle as Level 13's redirect method, extended to strip surrounding noise instead of just capturing clean stdout.

## LEVEL 17 — Diffing two files to find the changed line

**GOAL:** Compare passwords.old and passwords.new; find the one line that differs — that's the new password.

**CMD:**
```bash
  diff passwords.new passwords.old
```

**KEY CONCEPT:**

- diff shows line-by-line differences between two files.
- "<" = line from the FIRST file given, ">" = line from SECOND file.
- Format "42c42" means line 42 changed between both files.
- Real-world use: config file drift detection, comparing log snapshots, verifying file integrity after an incident.

## LEVEL 18 — Login that auto-terminates via shell startup files

**GOAL:** Read a file in bandit18's home, despite interactive SSH session getting killed immediately ("Byebye!") on login.

**CMD:**
```bash
  ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme" > output.txt
```

**KEY CONCEPT:**

- Passing a command directly to ssh (instead of just logging in) runs it without starting an interactive login shell, which avoided whatever in the startup file ("Byebye!") was killing the session on normal login. The exact mechanism depends on shell config, but bypassing the interactive shell was the fix here.

## LEVEL 19 — SETUID binary (privilege elevation via file permission)

**GOAL:** Use a setuid binary owned by bandit20 to read bandit20's password file, while logged in as bandit19.

**CMD:**
```bash
  ls -la                          → spot the "s" in owner permissions
  ./bandit20-do whoami            → confirms it runs AS bandit20
  ./bandit20-do cat /etc/bandit_pass/bandit20
```

**KEY CONCEPT:**

- setuid bit (shown as "s" instead of "x" in owner permissions, e.g. -rwsr-x---) makes a program execute with the FILE OWNER's privileges, not the invoking user's privileges.
- Only works for actual external executables — shell builtins (cd) and non-existent commands (setuid as a command) will fail since bandit20-do just launches a new process, it isn't a shell.
- /etc/bandit_pass is a DIRECTORY, not a file — always target the specific file inside it (/etc/bandit_pass/`<username>`).

## LEVEL 20 — SETUID binary that acts as a network client (suconnect)

**GOAL:** Use suconnect (setuid, runs as bandit20) to submit bandit20's password over a local TCP connection and receive bandit21's password in return.

**CMD:**
```bash
  Terminal 1 (listener — plays the "server" role):
    nc -l <port>

  Terminal 2 (client — the actual setuid binary):
    ./suconnect <same_port>

  Then type the current password INTO Terminal 1 (the listener),
  since suconnect reads whatever the listener sends and checks it.
```

**KEY CONCEPT:**

- suconnect only takes a port number as its argument — it doesn't take the password or any other command as an argument.
- Requires two simultaneous processes: one listening (nc -l), one connecting (suconnect) — same client/server pattern as Level 14, but here YOU must run both sides yourself instead of connecting to a pre-existing remote service.
- Trust the program's own confirmation message ("Password matches, sending next password") over an ambiguous terminal output — don't assume success just because some text appeared.

## LEVEL 21 — Cron job with insecure world-readable temp file

**GOAL:** Read the contents of /etc/cron.d/ entry for bandit22, then read the script it runs, then exploit the vulnerability it creates (password written to a world-readable /tmp file).

**CMD:**
```bash
  cat /etc/cron.d/cronjob_bandit22
  cat /usr/bin/cronjob_bandit22.sh
  cat /tmp/<filename_from_script>
```

**KEY CONCEPT:**

- Cron syntax: "* * * * * user /path/to/script" = runs every minute, as the specified user, running that script.
- @reboot = runs once at system startup instead of on a schedule.
- &> /dev/null discards all output (stdout+stderr), so you can't see results by running the script manually — you read the script source instead to understand its behavior.
- The vulnerability: the script does `chmod 644` on the temp file BEFORE writing the password into it, making it world-readable.
- Correct practice would be chmod 600 (owner-only) or not writing secrets to /tmp at all.

## LEVEL 22 — Cron job with dynamic (hash-based) filename

**GOAL:** Read the script behind bandit23's cronjob, understand its filename-generation logic, and manually replicate that logic to predict/find the output file — without executing the script yourself.

**CMD:**
```bash
  cat /etc/cron.d/cronjob_bandit23
  cat /usr/bin/cronjob_bandit23.sh
  echo I am user bandit23 | md5sum | cut -d ' ' -f 1
  cat /tmp/<resulting_hash>
```

**KEY CONCEPT:**

- The script runs automatically via cron AS bandit23 (not as you) — you never need to (and often can't) execute someone else's scheduled script directly; you just need to read its logic.
- $(whoami) inside the script resolves to whichever user cron invokes it as — since the cron line specifies "bandit23", that's what it evaluates to when it actually runs, even though YOUR whoami says bandit22.
- The filename is deterministic (md5 of a fixed string) — so it can be computed manually without ever running the target script.

## LEVEL 23 — Cron job executes YOUR script if you own it (privilege escalation via drop folder)

**GOAL:** Write a script that reads bandit24's password and writes it to a location you can read, then place it where bandit24's cron job will execute it automatically.

**CMD:**
```bash
  mktemp -d && cd <tempdir>
  nano myscript.sh
    → content:
      #!/bin/bash
      cat /etc/bandit_pass/bandit24 > /tmp/<random_unique_name>
  chmod +x myscript.sh
  cp myscript.sh /var/spool/bandit24/foo/
  (wait ~1 min for cron cycle)
  cat /tmp/<random_unique_name>
```

**KEY CONCEPT:**

- Drop-folder permission pattern: drwxrwx-wx on foo/ means you can WRITE into it but can't READ/LIST its contents — files placed there are "blind drops," verified only by their external effect.
- chmod +x MUST happen before cp — once inside foo/, you can't read/modify anything you placed there (no list/read access).
- Cron runs the script as bandit24, but ONLY if the script's owner matches the check in the parent script (owner="bandit23" check) — meaning ownership of the file (not just content) matters.
- The script self-deletes after execution (rm -rf in parent script) regardless of success/failure — no cleanup needed on your end.
- Never use placeholder syntax like `<name>` literally in shell code — < and > are redirect operators in bash, not generic brackets.

## LEVEL 24 — Brute-forcing a 4-digit PIN via single persistent connection

**GOAL:** Submit all 10,000 possible 4-digit PIN combinations (with bandit24's password) to a service on port 30002, in a single TCP connection, to find the one correct PIN.

**CMD:**
```bash
  for i in $(seq 0 9999); do printf "<password> %04d\n" $i; done | nc localhost 30002 > /tmp/result.txt
  grep -i "correct" /tmp/result.txt
```

**KEY CONCEPT:**

- printf "%04d" zero-pads a number to fixed width (0-9999 → 0000-9999).
- Piping the entire generated list into ONE nc connection (instead of reconnecting per guess) is what makes brute-forcing 10,000 combinations fast — matches the hint "no need to create new connections each time."
- Root-owned home directories block file creation there — write scratch/output files to /tmp/ instead.
- grep filters a huge noisy output down to just the line that matters.
