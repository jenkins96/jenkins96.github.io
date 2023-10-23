---
layout: post
title: Archiving And Compression In Linux
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [linux, archiving, compression]
comments: true
---
  
## Archiving - tar Command
The "tar" stands for tape archive. An archive file is a compressed file that contains one or more files bundled together for more accessible storage and portability.
  
* Archiving: combines multiples files into one.
  
The "tar" command can: create an archive, extract an archive, and list an archive (without needing to extract it first).
  
### Basics options
  
| Option | Description                                                                                                |
|--------|------------------------------------------------------------------------------------------------------------|
| -c     | Creates an archive by bundling files and directories together.                                             |
| -f     | Specifies the filename of the archive to be created or extracted.                                          |
| -v     | Verbose information.                                                                                       |
| -t     | List the contents of an archive.                                                                           |
| -x     | Extract files from an archive.                                                                             |
| -z     | Compress with gzip. Extension ".tar.gz".                                                                   |
| -j     | Compress with bzip2. Extension ".tar.bz2".                                                                 |
| -xz    | Compress with xz. Extension ".tar.xz".                                                                     |
| -u     | Archives and adds new files or directories to an existing archive.                                         |
| -r     | Updates or adds files or directories to an already existing archive without recreating the entire archive. |
  
### Archiving Whole Directory
  
```
$ tar -cvf LinuxBooks.tar LinuxBooks/
LinuxBooks/
LinuxBooks/fhs-3.0.pdf
LinuxBooks/Bash-Beginners-Guide.pdf
LinuxBooks/intro-linux.pdf
LinuxBooks/GNU-Linux-Tools-Summary.pdf
LinuxBooks/linuxfun.pdf
LinuxBooks/learn-linux-in-5-days.pdf
$ ls
file1.txt  file2.txt  image1.jpg  image2.jpg  LinuxBooks  LinuxBooks.tar
```
  
### Archiving Specific Files
  
```
$ tar -cvf my_archive.tar image1.jpg file1.txt 
image1.jpg
file1.txt

$ ls
file1.txt  image1.jpg  LinuxBooks      my_archive.tar
file2.txt  image2.jpg  LinuxBooks.tar
```
  
### Archiving All .jpg Files
  
```
$ tar -cvf only_jpg.tar *.jpg
image1.jpg
image2.jpg

$ ls
file1.txt  image1.jpg  LinuxBooks      my_archive.tar
file2.txt  image2.jpg  LinuxBooks.tar  only_jpg.tar
```
  
### Listing An Archive
One of the actions that "tar" command can do, is to just list what is inside a archive file without extracting the whole content.
  
Use "-t" option.
  
```
$ tar -tvf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt

```
  
### Updating New Files To Archive Without Recreating Archive
Use "-r" option.
  
```
$ tar -tvf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt

$ tar -rvf my_archive.tar file2.txt 
file2.txt

$ tar -tvf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt
-rw-rw-r-- adrian/adrian       85 2023-10-22 09:43 file2.txt
```
  
### Archives And Adds New Files To Archive
Use "-u" option.
  
```
$ tar -tvf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt
-rw-rw-r-- adrian/adrian       85 2023-10-22 09:43 file2.txt

$ tar -uvf my_archive.tar image2.jpg 
image2.jpg

$ tar -tvf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt
-rw-rw-r-- adrian/adrian       85 2023-10-22 09:43 file2.txt
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image2.jpg
```
  
### Extract Archive In Same Directory
Use "-x" option to extract.
  
I have an archive of "LinuxBooks/" direcotry.
I will delete directory and then, extract the contents of my archive.
  
```
$ ls
file1.txt  image1.jpg  LinuxBooks           LinuxBooks.tar  only_jpg.tar
file2.txt  image2.jpg  LinuxBooksExtracted  my_archive.tar

$ rm -R LinuxBooks

$ ls
file1.txt  image1.jpg  LinuxBooksExtracted  my_archive.tar
file2.txt  image2.jpg  LinuxBooks.tar       only_jpg.tar

$ tar -xvf LinuxBooks.tar
LinuxBooks/
LinuxBooks/fhs-3.0.pdf
LinuxBooks/Bash-Beginners-Guide.pdf
LinuxBooks/intro-linux.pdf
LinuxBooks/GNU-Linux-Tools-Summary.pdf
LinuxBooks/linuxfun.pdf
LinuxBooks/learn-linux-in-5-days.pdf

$ ls
file1.txt  image1.jpg  LinuxBooks           LinuxBooks.tar  only_jpg.tar
file2.txt  image2.jpg  LinuxBooksExtracted  my_archive.tar
```
  
### Extracting Archive To Specific Location
Use "-x" option to extract.
  
Use "-C" to specify location to extract.
  
```
$ tar -xvf LinuxBooks.tar -C LinuxBooksExtracted/
LinuxBooks/
LinuxBooks/fhs-3.0.pdf
LinuxBooks/Bash-Beginners-Guide.pdf
LinuxBooks/intro-linux.pdf
LinuxBooks/GNU-Linux-Tools-Summary.pdf
LinuxBooks/linuxfun.pdf
LinuxBooks/learn-linux-in-5-days.pdf

$ ls LinuxBooksExtracted/
LinuxBooks
```
  
### Extracting Specific Files From Archive
Seems to be order matters. Option "-C" cannot be at the end.
   
```
$ tar -tf only_jpg.tar 
image1.jpg
image2.jpg

$ tar -C otherExtractions/ -xvf only_jpg.tar "image1.jpg" 
image1.jpg

$ ls otherExtractions/
image1.jpg
```
  
### Deleting File From Archive
Use the "--delete" option.

```
$ tar -vtf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt
-rw-rw-r-- adrian/adrian       85 2023-10-22 09:43 file2.txt
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image2.jpg

$ tar --delete -f my_archive.tar  file2.txt 

$ tar -vtf my_archive.tar 
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image1.jpg
-rw-rw-r-- adrian/adrian       29 2023-10-22 09:43 file1.txt
-rw-rw-r-- adrian/adrian 10771425 2023-10-14 11:15 image2.jpg
```
  
## Compression

* Compression:makes the files smaller by removing redundant information.

Types of compression:
* **Lossless**: no information is removed from the file. Examples:PNG, ZIP, BMP.
* **Lossy**: information is permanently removed from file. However, this information is unnoticeable for us humans.Mainly used for images, videos, audios. Examples: JPEG, MP3, WMA.

### gzip/gunzip

Uses "Lempel-Ziv" algorithm.

**Compress**:
  
```
$ ls
file1.txt  file2.txt  file3.txt  image1.jpg  image2.jpg  LinuxBooks

$ gzip file3.txt

$ ls
file1.txt  file2.txt  file3.txt.gz  image1.jpg  image2.jpg  LinuxBooks
```
  
The only problem here is that it replaces original file. But we can use the "-k" option to avoid this:
  
```
$ ls
file1.txt  file2.txt  file3.txt.gz  image1.jpg  image2.jpg  LinuxBooks

$ gzip -k file2.txt 

$ ls
file1.txt  file2.txt  file2.txt.gz  file3.txt.gz  image1.jpg  image2.jpg  LinuxBooks
```
  
We cannot compress direcotories, but we "-r" option we can compress everything inside a direcotry:
  
```
$ gzip -r LinuxBooks/

$ ls /LinuxBooks/
Bash-Beginners-Guide.pdf.gz  GNU-Linux-Tools-Summary.pdf.gz  learn-linux-in-5-days.pdf.gz
fhs-3.0.pdf.gz               intro-linux.pdf.gz              linuxfun.pdf.gz
```
  
We can use option "1-9" to change compression level, where 1 is the lowest and 0 is the highest compression.
  
```
$ ls -lh
total 266M
-rw-rw-r-- 1 adrian adrian 266M oct 17 05:53 video.mp4

$ gzip -k video.mp4 

$ gzip -l video.mp4.gz 
         compressed        uncompressed  ratio uncompressed_name
          221946210           278535353  20.3% video.mp4

$ gzip -k -9 video.mp4 
gzip: video.mp4.gz already exists; do you wish to overwrite (y or n)? y

$ gzip -l video.mp4.gz 
         compressed        uncompressed  ratio uncompressed_name
          221842374           278535353  20.4% video.mp4


$ gzip -k -1 -v video.mp4 
gzip: video.mp4.gz already exists; do you wish to overwrite (y or n)? y
video.mp4:	 19.6% -- created video.mp4.gz

$ gzip -l video.mp4.gz 
         compressed        uncompressed  ratio uncompressed_name
          224047593           278535353  19.6% video.mp4
```
  
To decompress we can use either "gzip -d file.gz" or "gunzip", same thing.
Same thing, use "-k" option to preserve the ".gz" extension.
  
```
$ ls
file1.txt.gz  file2.txt.gz  folder      image2.jpg
file2.txt     file3.txt.gz  image1.jpg  LinuxBooks

$ gzip -d file1.txt.gz 

$ ls
file1.txt  file2.txt  file2.txt.gz  file3.txt.gz  folder  image1.jpg  image2.jpg  LinuxBooks

$ gzip -d -k file3.txt.gz 

$ ls
file1.txt  file2.txt.gz  file3.txt.gz  image1.jpg  LinuxBooks
file2.txt  file3.txt     folder        image2.jpg

# gunzip
$ ls
video.mp4.gz

$ gunzip video.mp4.gz 

$ ls
video.mp4
```
  
### bzip2/bunzip2
Uses "Burrows-Wheeler" algorithm.
  
```
$ bzip2 file1.txt 

$ ls
file1.txt.bz2  file2.txt  file3.txt  folder  image1.jpg  image2.jpg  LinuxBooks

$ bunzip2 file1.txt.bz2 

$ ls
file1.txt  file2.txt  file3.txt  folder  image1.jpg  image2.jpg  LinuxBooks
```

### xz/unxz
  
```
$ xz file3.txt 

$ ls
file1.txt  file2.txt  file3.txt.xz  folder  image1.jpg  image2.jpg  LinuxBooks

$ unxz file3.txt.xz 
adrian@vivobook:~/Documents/testing$ ls
file1.txt  file2.txt  file3.txt  folder  image1.jpg  image2.jpg  LinuxBooks
```
  
### zip/unzip
I think this is the only one capable of actually compression whole folder.
  
```
$ zip myfile3.txt.zip file3.txt 
  adding: file3.txt (deflated 58%)

$ ls
file1.txt  file2.txt  file3.txt  folder  image1.jpg  image2.jpg  LinuxBooks  myfile3.txt.zip

$ unzip myfile3.txt.zip 
Archive:  myfile3.txt.zip
replace file3.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
  inflating: file3.txt               

$ ls
file1.txt  file2.txt  file3.txt  folder  image1.jpg  image2.jpg  LinuxBooks  myfile3.txt.zip

```
  
## Combining Archiving & Compression
  
| Command                                         | Description                                  |
|-------------------------------------------------|----------------------------------------------|
| tar -zcf  <file.tar.gz>   <directory/file>  | Creates tar file and compress it with gzip.  |
| tar -xzcf  <file.tar.gz>   <directory/file> | Creates tar file and compress it with xz.    |
| tar -jcf  <file.tar.gz>   <directory/file>  | Creates tar file and compress it with bzip2. |

## Resources
* [Compressing files under Linux or UNIX cheat sheet](https://www.cyberciti.biz/howto/question/general/compress-file-unix-linux-cheat-sheet.php#bzip2)
* [Gzip Command in Linux](https://www.geeksforgeeks.org/gzip-command-linux/)
* [tar Command](https://www.geeksforgeeks.org/tar-command-linux-examples/)
* [Linux tar Command](https://www.tutorialspoint.com/linux-tar-command)
* [Tar in Linux – Tar GZ, Tar File, Tar Directory, and Tar Compress Command Examples](https://www.freecodecamp.org/news/tar-in-linux-example-tar-gz-tar-file-and-tar-directory-and-tar-compress-commands/)
* [How to Use the gzip Command in Linux](https://linuxhandbook.com/gzip-command/)




