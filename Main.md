# TryHackMe - Lian_Yu WriteUp

## Tools used:

1. Nmap
2. Dirsearch
3. Gobuster
4. Steghide
5. hexeditor
6. GTFOBins

## Scanning and Enumeration

NOTE: The IP will be displayed as **"10.10.x.x"** due to the different IP's that TryHackMe gives

Run 3 Nmap scans

### Half-TCP scan

```
nmap -sS -vv -T5 -n -p 1-20000 10.10.x.x -oN Half-TCP.txt
```

### Version scan

```
nmap -sV --version-all -vv -T5 -p 21,22,80,111 10.10.x.x -oN Version-scan.txt
```

### Aggressive scan

```
nmap -A -vv -T5 10.10.x.x -oN Aggressive.txt
```

It's a good practice to always scan the machine even though you get the same results everytime, maybe you can find a vulnerable service  

The output of the three scans is what you'll use to your advantage

## Enumeration of the website

Prepare to scan for a long time, this part kinda took me 1-2 hours to do.  
Go to the website hosted on port 80 then scan it using either Dirsearch or Gobuster

Here are the commands below:

### Dirsearch

```
dirsearch -u http://10.10.x.x -t 120 -o Initial-Directory-scan
```

### Gobuster

```
gobuster dir -u http://10.10.x.x -w /path/to/wordlist -X /path/to/extension/wordlist -t 120 -o Initial-Directory-scan
```

After the scans complete, you'll see 1 accessible directory andd go to that directory.  
After going to that directory, highlight the text below the paragraphs and you'll see a **codename**

Next, scan the directory you're in again using the same command above but you may have to use a different directory wordlist.

After the scans complete again, you should see a directory with numbers as it's name.  Don't go down the rabbit hole and inspect the HTML source code and you'll see a **extension** commented out.

Modify the commands above to look only for the extension.

### Dirsearch

```
dirsearch -u http://10.10.x.x/island/2100 -t 120 -o Ticket-extension-scan -e ticket
```

### Gobuster

```
gobuster dir -u http://10.10.x.x/island/2100 -w /path/to/wordlist -x ticekt -t 120 -o Ticket-extension-scan
```

After these scans, you should see a accessible file, access the file and extract the encoded text.

## Decoding Base58

The encoded text that you've extracted is encoded in base58, look up a online decoder and decode it and extract the password.

## Attacking FTP, Steganography and Hex editing

Remember the highlighted text? Log in to the FTP server using the highlighted text you found and the base58 decoded text.  

Once your inside the server, extract everything including the two hidden files.  
I believe the file's name were **.other_user.txt** and **.profile**.

Once you've extracted everything, open the "Leave_me_alone.png" file using hexeditor.

```
hexeditor Leave_me_alone.png
```

Search online the hex value for the acronym of PNG, then modify the first line to have the PNG header.  
If you've done everything right and when you open the image, you should see a password **highlighted in red**, extract it.

Use the extracted password alongside steghide.  
Use the command below to extract the hidden contents in the **aa.jpg** file.

```
steghide extract -sf aa.jpg
```

It will prompt you for a password, provide the password you found when you were modifying the hex of the image.

If done right, you should see two files: **"passwd.txt"** and **"shado"**
The **"passwd.txt"** will provide you the name of the other use and the **"shado"** file will provide you the password to SSH.

## Attacking SSH and Privilege Escalation

Use the username you found on the **"passwd.txt"** file and the password from the **"shado"** file and login to SSH

After logging in to SSH, immediately extract the user flag and submit it to TryHackMe.

Run the command below:

```
sudo -l
```

The commandd above will tell you that Slade, the user you're logged in as, can run pkexec with root privileges.

pkexec is a program that allows a user to execute a program as another user.  
Go to GTFOBins and search for the pkexec binary exploit, paste it in the terminal and after that you should have root privileges.

Extract the root flag and submit it to TryHackMe and you have just successfully pawned the TryHackMe Lian_Yu box
