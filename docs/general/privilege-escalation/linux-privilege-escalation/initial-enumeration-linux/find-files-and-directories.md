# Find Files and Directories

### which

`which <PROGRAM>`: This tool returns the path to the file or link to be executed. This allows us to determine if specific programs, such as cURL, netcat, wget, python, gcc, are available on the operating system.

### find

```bash
find <PATH> <ARGUMENTS>
find / -type f -name <NAME> 2>/dev/null # Search for files by a given name.
find / -type d -name <NAME> 2>/dev/null # Search directories by a given name.
find / -type f -user root # Search for files belonging to the root user.
find / -type f -group root # Search for files belonging to the root group.
# Advanced
find / -type f -size +20k # Find files that are larger than 20 kilobytes (c=bytes,k=kilobytes,m=megabytes,g=giga)
find / -type f -size -20k # Find files that are less than 20 kilobytes in size.
find / -type f -size 20k # Search for files that have a size of 20 kilobytes.
find / -type f -name "user.txt" -exec ls -l {} \; 2>/dev/null # Search for the user.txt that if found, execute a ls -l
find / -type f -exec grep hello {} \; 2>/dev/null # Search for files that have the word "hello" in them
find / -type f -exec grep -H hello {} \; # Search for files that have the word "hello" in them and display the filenames.
find / -type f -exec grep hello {} + # Search for files that have the word "hello" in them and display the filenames.
find / -type f -exec grep -H '^passwd' {} \; # Search for files that have in their contents a line beginning with: passwd
find / -type f -newermt 2020-03-03 # With this option, we set the date. Only files newer than the specified date will be presented.
find / -type f -newermt '6/30/2020 0:00:00' # All dates/times after 6/30/2020 0:00:00 will be considered as a condition to search for)
find / -type f -newerat 2017-09-12 ! -newerat 2017-09-09-14 # All dates before 2017-09-12 will be excluded; all dates after 2017-09-14 will be excluded, so only 2017-09-13 remains as a date to search).
find / -type f -atime -7 # Files not accessed in the last 7 days
find / -type f -mtime -1 # Files that were not modified in the last 7 days
find / -type f -ctime -7 # Files that were modified in the last 7 days
find / -type f -mmin -120 # Files that were modified in the last 2 hours (120 minutes)
# Others
find / -readable 2>/dev/null # Find all files that we can read.
find / -writable 2>/dev/null # Find all files that we can write/modify.
find / -executable 2>/dev/null # Find all files we can execute.
```

<details>

<summary>Parameters</summary>

* `-name <NAME>` : Search for a specific name. We can include asterisks. Example: `find / -name '*pass*' 2>/dev/null`.
* `-type <TYPE>` : Specify the type of object. The most common are: `f` for file, `d` for directory, `l` for symbolic link and `s` for socket.
* `-size <PREFIX><NUM>n[cwbkMG]`
  * `<PREFIX>` : Can be a `+` (greater than this much size) or a `-` (less than this much size).
  * `<NUM>` : Number.
  * `n[cwbkMG]` : Specify the size type. The most usual will be `c` (bytes).

</details>

### locate

It will take us a long time to search the entire system for our files and directories to perform many different searches. The `locate` command offers us a faster way to search the system. Unlike the `find` command, `locate` works with a local database that contains all the information about existing files and folders. We can update this database with the following command.

```
sudo updatedb
```

If we now search for all files with the extension ".conf", you will find that this search produces results much faster than using find.

However, this tool does not have as many filtering options that we can use. So it is always worth considering whether we can use the locate command or use the find command instead. It always depends on what we are looking for.

```
locate *.conf
```
