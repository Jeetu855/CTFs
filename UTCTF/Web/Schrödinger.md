
We are allowed to upload zip files
I am going to explain how zip symlinks exploit

#### First of all what are symlinks?

A symlink (also called a symbolic link) is a type of file in Linux that points to another file or a folder on your computer. Symlinks are similar to shortcuts in Windows.

There are two types of symlinks, i) Soft symlink , ii) Hard Symlink

A symbolic or soft link is an actual link to the original file, whereas a hard link is a mirror copy of the original file. If you delete the original file, the soft link has no value, because it points to a non-existent file. But in the case of hard links, it is entirely the opposite. Even if you delete the original file, the hard link will still have the data of the original file. Because hard link acts as a mirror copy of the original file

#### Zip Symlink Vulnerablity

An  archive can contain a symbolic link. A symbolic link is a special file that links to another file. By uploading a zip containing a symbolic link, and after the zip is extracted, you can access the symbolic link to gain access to files that you should not get access to. To do so, you need to get your symbolic link to point to files outside of the web root, for example /etc/passwd.

These type of issues are typically found when a developer allowing to accept zip file in our upload functionality. When a user uploads the zip file in the application then it  simply takes the zip file and extracts its files without any validations.

First create a symlink of /etc/passwd file using the following command

```sh
ln -s /etc/passwd etc
```

This create a symlink for /etc/passwd file and the file that points to it is called its symlink in this case its name is `etc`

![[Pasted image 20240401162352.png]]

This is how it looks like if you run the `ls -la` command

Now zip this file using the `zip` command

The thing is when you use `zip` to compress files and directories, it includes the symlinks themselves rather than following them and including the files they point to.
However, if you want to include the files pointed to by symbolic links in the zip archive, you have to use the `--symlinks`

```sh
zip -r --symlinks file.zip etc 
```

I am naming the zip file as file.zip

Now upload this file and observe the results

![[Pasted image 20240401162856.png]]

It prints out the /etc/passwd file 

From the description we know that the flag is in /home/user/flag.txt
We get the username from /etc/passwd file as `copenhagen`

So first create a directory named `home` , inside it create a directory named `copenhagen` and inside this directory create a file named `flag.txt`

Now run 
```sh
ln -s /home/copenhagen/flag.txt flag
```

A thing to note here is that when you try to press tab to auto-complete it wont because we are doing `/home/copenhagen` but we dont actually have a user named `copenhagen` on our machine . Your  default `home` directory will contain directories for user on your machine. But think about the target system for a second. I says that flag is in `/home/copenhagen/flag.txt` , so from that point of view, what we are doing is correct.

Now zip archive this symlink that we created for `flag.txt`

```sh
zip -r --symlinks flag.zip flag
```

Now upload the `flag.zip` file and we get this flag.

flag : utflag{No_Observable_Cats_Were_Harmed}
