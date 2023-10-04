

## Using find Command In Linux
This command is used to find files/directories in a UNIX like systems.
  
It searches the directory tree rooted at each given starting-point by evaluating the given expression from left to right. In other words, it help us find files or directories based on size, name, permissions, last access,etc.
  
Here is the syntax:
```
find [OPTIONS] [starting point for the search] [expression]
```
  
* **OPTIONS**: refers to behavior regarding symbolic links. Not going to address these options here. Feel free to read man pages.
* **Starting-point**: actual starting point from where you want to beging searching.
* **Expression**: in here we actually specify how we match files and what to do with the files that are matched. There are a log of options to choose here. We will explore some, but do take a look at documentation.
  
### Find Specific File Name
Let us start with an easy one.
  
Suppose I know the name of the file and I have some idea where it might belocated. So, I know file name is "findme" and I know it is located somewhere in "/home/adrian" directory.
  
```
$ find /home/adrian -name "findme"
/home/adrian/Downloads/findme
```
  
Great, I now know it is located in "/home/adrian/Downloads".
  
### Find By File Name(Case Insensitive)
We can use "-iname" option to ignore case sensitivity.
  
```
$ find /home/adrian -iname "canyoufindme"
/home/adrian/canyoufindme
/home/adrian/CANYOUFINDME
```
  
### Find All ".txt" File Extension
Now I want to locate all the files inside my home directory that have ".txt" file extension. I will only show the first three results.
  
```
$ find /home/adrian -name "*.txt" | head -n 3
/home/adrian/docker_test/welcome-to-docker/public/robots.txt
/home/adrian/.oh-my-zsh/LICENSE.txt
/home/adrian/.oh-my-zsh/plugins/golang/templates/package.txt
```
  
### Find By Directory/Regular File/Symbolic Link
With the "-type" option we can seach for directory, regular file, symbolic link, amont others.
  
* d = directory.
* f = regular file.
* l = symbolic link.
  
Finding symbolic link:
  
```
find /home/adrian/Desktop/ -type l
/home/adrian/Desktop/text_file
```
  
Finding a directory called "Tasks" in whole file system. This requires sudo privilages to do a proper search. Also, I will send standard error to /dev/null because I do not want to see it.
  
```
$ sudo find / -name "Tasks" -type d 2> /dev/null
/home/adrian/Documents/Tasks

```
  
### Find Directories & Subdirectories
```
$ find . -type d
.
./welcome-to-docker
./welcome-to-docker/public
./welcome-to-docker/src
./welcome-to-docker/.git
./welcome-to-docker/.git/hooks
./welcome-to-docker/.git/refs
./welcome-to-docker/.git/refs/remotes
./welcome-to-docker/.git/refs/remotes/origin
./welcome-to-docker/.git/refs/tags
./welcome-to-docker/.git/refs/heads
./welcome-to-docker/.git/objects
./welcome-to-docker/.git/objects/info
./welcome-to-docker/.git/objects/pack
./welcome-to-docker/.git/info
./welcome-to-docker/.git/branches
./welcome-to-docker/.git/logs
./welcome-to-docker/.git/logs/refs
./welcome-to-docker/.git/logs/refs/remotes
./welcome-to-docker/.git/logs/refs/remotes/origin
./welcome-to-docker/.git/logs/refs/heads

```
  
### Find & Empty Content
We can even use the "-empty" option to search for, well, empty content.
  
```
$ find /home/adrian/Documents/doIHaveEmptycontent/ -empty
/home/adrian/Documents/doIHaveEmptycontent/empty
```
  
If we actually take a look at what we have in that folder there is an empty file and another file with 11bytes of content:
  
```
$ wc -c /home/adrian/Documents/doIHaveEmptycontent/empty 
0 /home/adrian/Documents/doIHaveEmptycontent/empty
$ wc -c /home/adrian/Documents/doIHaveEmptycontent/notempty 
11 /home/adrian/Documents/doIHaveEmptycontent/notempty

```
  
### Find & Deleting Empty Content
The "find" command also has "actions". Based on the result of "find" command, if there is a match or not, you can choose an action to be perform against all files or folders that matched your expression. Among the actions you may take we have "delete" and "exec"; latter meaning  you could specify a custom command to run if "find" command is successful.
    
Say you wanted to delete empty content inside above directory. We can use the "-delete" option. However, we need to be careful. If set at wrong place, it could delete unwanted files.
  
```
$ ls --size
total 4
0 empty  4 notempty
```
  
```
$ find /home/adrian/Documents/doIHaveEmptycontent/ -empty -delete
```
  
```
$ ls --size
total 4
4 notempty
adrian@vivob
```
  
Do take care about order, in this case we told to "find" empty content in that location, this returns "a list" of empty file(s), then, delete file(s) returned in that list. If we would have specified "-delete" first and then "-empty", we would have actually deleted whole folder given that we first are asking to delete and then search for empty content.
  
### Find & Change Permissions 
Now, let us suppose I want to change all permissions for all files that matches a criteria. All my files inside "~/Picures/Screenshots" have 664 permission. This is user & group: rw, while others: r. My goal isto change all ".png" to have 700 permission. 
  
```
$ ls -l
total 304
-rw-rw-r-- 1 adrian adrian  61756 jun  5 12:27 'Screenshot from 2023-06-05 12-27-26.png'
-rw-rw-r-- 1 adrian adrian 112578 jun  6 16:08 'Screenshot from 2023-06-06 16-08-26.png'
-rw-rw-r-- 1 adrian adrian  43324 sep 28 19:16 'Screenshot from 2023-09-28 19-16-17.png'
-rw-rw-r-- 1 adrian adrian  85566 sep 30 18:17 'Screenshot from 2023-09-30 18-17-11.png'
-rw-rw-r-- 1 adrian adrian      0 oct  1 14:50  test.txt
```
  
```
$ find /home/adrian/Pictures/Screenshots/ -name "*.png" -exec chmod 700 {} +
```
  
```
$ ls -l
total 304
-rwx------ 1 adrian adrian  61756 jun  5 12:27 'Screenshot from 2023-06-05 12-27-26.png'
-rwx------ 1 adrian adrian 112578 jun  6 16:08 'Screenshot from 2023-06-06 16-08-26.png'
-rwx------ 1 adrian adrian  43324 sep 28 19:16 'Screenshot from 2023-09-28 19-16-17.png'
-rwx------ 1 adrian adrian  85566 sep 30 18:17 'Screenshot from 2023-09-30 18-17-11.png'
-rw-rw-r-- 1 adrian adrian      0 oct  1 14:50  test.txt
```
  
Notice all ".png" changed from "rw-rw-r--" to "rwx------", while the "test.txt" stayed the same.
  
So:
* Find everything with ".png" extension in that directory and put it in our "return list".
* Execute given command to all files that matched criteria and were added to our "return list".
* {} is syntax. You can read more about it in the man pages.
* + must end with plus sign (+) or espaced character (\;) to protect from interporetation by the shell.
  
### Find Files Owned By Specific User
Let us search for ".png" files owned by user "adrian". For this we can use option "-user".
  
```
$ find ~ -name "*.png" -user "adrian" | head -n 3
/home/adrian/.oh-my-zsh/plugins/zsh-navigation-tools/doc/img/n-history2.png
/home/adrian/.oh-my-zsh/plugins/catimg/colors.png
/home/adrian/.oh-my-zsh/plugins/kubectx/stage.png
```
  
### Find by Content
We want to locate files that contain word "paul!" in ".txt" files. Here we can first use "find" command to locate text files and then use "grep" command for string matching.
  
```
$ find /home/adrian -name "*.txt" -exec grep -iH "paul!" {} +
/home/adrian/Documents/test3.txt:This is Paul!
/home/adrian/Documents/test1.txt:This is Paul!
```
    
The "-iH" for "grep" command stands for ignore case (i) and print file name of each match (H).

### Find By Path
We can use the "-ipath" option for this.
  
```
$ find / -type d -ipath "*to-docker/src" 2> /dev/null
/home/adrian/docker_test/welcome-to-docker/src
```
  
### Find Files With Permission 700
  
```
$ find /home/adrian/Pictures -perm 700
/home/adrian/Pictures/Screenshots/Screenshot from 2023-09-28 19-16-17.png
/home/adrian/Pictures/Screenshots/Screenshot from 2023-06-06 16-08-26.png
/home/adrian/Pictures/Screenshots/Screenshot from 2023-09-30 18-17-11.png
/home/adrian/Pictures/Screenshots/Screenshot from 2023-06-05 12-27-26.png
```
  
### Find Files Larger Than X Bytes
Here we can use the "-size" option.
  
Supported units:
* b = for 512-byte blocks (default).
* c = bytes.
* w = two-byte words.
* k = Kibibytes (1024 bytes).
* M = mebibytes (1048576 bytes).
* G = gibibytes (1073741824 bytes).
  
```
$ find ~/Pictures/Screenshots/ -size +81920c
/home/adrian/Pictures/Screenshots/Screenshot from 2023-06-06 16-08-26.png
/home/adrian/Pictures/Screenshots/Screenshot from 2023-09-30 18-17-11.png
```
  
### Find Multiple Files With Different Extensions
  
```
$ find . -type f -name "*.txt" -o -name "*.doc" | head -n 3
./file.doc
./Tasks/Cases/case_2551_closed.txt
./Tasks/Cases/case_2636_closed.txt
```
  
### Find Files Modified Within Last 10 Minute
  
```
$ find . -amin -10 -type f
./file.doc
./file.pdf
```
  
### Find .log Files and Truncate Them
  
Command will find all files ending in ".log" and it will truncate them to a size of 0.
  
```
$ sudo find /var/log -type f -name *.log -exec truncate -s 0 {} +
```
  
```
$ ls --size -l /var/log | grep '\.log$'
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 alternatives.log
    0 -rw-r-----  1 root              adm                     0 oct  3 13:20 apport.log
    4 -rw-r-----  1 syslog            adm                   284 oct  3 13:25 auth.log
    0 -rw-------  1 root              root                    0 oct  3 13:20 boot.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 bootstrap.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 dpkg.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 fontconfig.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 gpu-manager.log
    0 -rw-r-----  1 syslog            adm                     0 oct  3 13:20 kern.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 macchanger.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 ubuntu-advantage.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 ubuntu-advantage-timer.log
    0 -rw-r--r--  1 root              root                    0 oct  3 13:20 vbox-setup.log

```
  
## Get GitHub Token Because I Forgot Where It Was
:)  
  
```
$ find /home/adrian -name "github_token" -exec cat {} +
<TOKEN VALUE>
```
  

## Resources
* [35 Practical Examples of Linux Find Command](https://www.tecmint.com/35-practical-examples-of-linux-find-command/)
* [find command in Linux with examples](https://www.geeksforgeeks.org/find-command-in-linux-with-examples/)
* [10 ways to use the Linux find command](https://www.redhat.com/sysadmin/linux-find-command)
* [40 Best Examples of Find command in Linux ](https://geekflare.com/linux-find-commands/)