# Simple Notes for Basic Linux Commands

These notes are based on the GeeksforGeeks Linux commands tutorial.

---

## 1. `ls` — List Files and Folders
Shows files and directories in the current folder.

### Syntax
```bash
ls
```

### Useful Options
```bash
ls -l     # detailed view
ls -a     # show hidden files
ls -lh    # human readable sizes
```

### Example
```bash
ls Documents
```

---

## 2. `pwd` — Present Working Directory
Displays current directory path.

### Syntax
```bash
pwd
```

### Example
```bash
/home/manasvini
```

---

## 3. `mkdir` — Create Directory
Creates a new folder.

### Syntax
```bash
mkdir folder_name
```

### Example
```bash
mkdir projects
```

---

## 4. `cd` — Change Directory
Moves between folders.

### Syntax
```bash
cd directory_name
```

### Examples
```bash
cd Documents
cd ..          # go back one folder
cd ~           # go to home directory
```

---

## 5. `rmdir` — Remove Empty Directory
Deletes empty folders only.

### Syntax
```bash
rmdir folder_name
```

### Example
```bash
rmdir test
```

---

## 6. `cp` — Copy Files/Folders
Copies files from one place to another.

### Syntax
```bash
cp source destination
```

### Example
```bash
cp file1.txt backup.txt
```

### Copy Folder
```bash
cp -r folder1 folder2
```

---

## 7. `mv` — Move or Rename
Moves files/folders or renames them.

### Syntax
```bash
mv old_name new_name
```

### Examples
```bash
mv file.txt newfile.txt
mv file.txt /Documents
```

---

## 8. `rm` — Remove Files
Deletes files permanently.

### Syntax
```bash
rm file_name
```

### Examples
```bash
rm demo.txt
rm -r foldername     # remove folder
rm -f file.txt       # force delete
```

⚠️ Be careful: deleted files usually cannot be recovered.

---

## 9. `touch` — Create Empty File
Creates a new empty file.

### Syntax
```bash
touch filename
```

### Example
```bash
touch notes.txt
```

---

## 10. `cat` — Display File Content
Shows contents of a file.

### Syntax
```bash
cat filename
```

### Example
```bash
cat notes.txt
```

---

## 11. `nano` — Text Editor
Opens terminal-based text editor.

### Syntax
```bash
nano filename
```

### Example
```bash
nano notes.txt
```

### Shortcuts
```bash
Ctrl + O   # save
Ctrl + X   # exit
```

---

## 12. `grep` — Search Text
Searches for words/patterns in files.

### Syntax
```bash
grep "word" filename
```

### Example
```bash
grep "hello" notes.txt
```

---

## 13. `find` — Search Files/Folders
Finds files in directories.

### Syntax
```bash
find path -name filename
```

### Example
```bash
find . -name notes.txt
```

---

## 14. `locate` — Quickly Find Files
Searches files using database.

### Syntax
```bash
locate filename
```

### Example
```bash
locate notes.txt
```

---

## 15. `chmod` — Change Permissions
Changes file permissions.

### Syntax
```bash
chmod permissions file
```

### Example
```bash
chmod 777 file.txt
```

### Common Permissions

| Number | Meaning |
|---|---|
| 7 | read + write + execute |
| 6 | read + write |
| 5 | read + execute |
| 4 | read only |

---

## 16. `chown` — Change Ownership
Changes file owner.

### Syntax
```bash
chown user file
```

### Example
```bash
chown manasvini file.txt
```

---

## 17. `uname` — System Information
Shows OS details.

### Syntax
```bash
uname
```

### More Info
```bash
uname -a
```

---

## 18. `sudo` — Run as Administrator
Executes commands with root privileges.

### Syntax
```bash
sudo command
```

### Example
```bash
sudo apt update
```

---

## 19. `apt-get` — Package Manager
Installs and manages software (Ubuntu/Debian).

### Install Package
```bash
sudo apt-get install package_name
```

### Update Packages
```bash
sudo apt update
```

---

## 20. `wget` — Download Files
Downloads files from internet.

### Syntax
```bash
wget URL
```

### Example
```bash
wget https://example.com/file.zip
```

---

# Quick Cheat Sheet

| Command | Purpose |
|---|---|
| `ls` | List files |
| `pwd` | Show current directory |
| `cd` | Change directory |
| `mkdir` | Create folder |
| `rmdir` | Remove empty folder |
| `touch` | Create file |
| `cp` | Copy files |
| `mv` | Move/Rename files |
| `rm` | Delete files |
| `cat` | View file content |
| `nano` | Edit files |
| `grep` | Search text |
| `find` | Find files |
| `chmod` | Change permissions |
| `sudo` | Admin access |

Source:
https://www.geeksforgeeks.org/linux-unix/basic-linux-commands/
