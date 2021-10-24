# LAB1 Q&A


## Part 1

### 1.  What is /etc/apt/sources.list.d directory stands for? How can you use it? Provide an example.

To start with, it is better first to describe `/etc/apt/sources.list` file. This file contains a list of repositories (package sources) of the system, where each line is a package info (source, format, and other components). That file is *unique* and usually it contains system native repositories. Thus, imagine we have a lot of repositories and we want to remove one of them. For that, we need to manipulate the whole list. This causes access issues (with only one file we would have many users to have access to it) and, consequently, harder to maintain. With `etc/apt/sources.list.d` is a directory, users can create multiple files of .list and define repositories there. As a result, when one wants to remove a repository, they can simply remove specific file from `/etc/apt/sources.list.d` without manipulating the main list. This is much more manageable. 

#### Usage example: 

I would use `nano /etc/apt/sources.list.d/filename.list`, because I want to add a repository definition in a separete file, then I would add a line (its format is specified in the second task) and save the file. 


### 2.  How can you add/delete 3rd party repositories to install required software? Provide an example.

#### Adding:
As discussed in the previous question, in order to add a 3rd party repository we need to define this repository inside `/etc/apt/sources.list` or in a separate file inside `/etc/apt/sources.list.d` directory. The format of the line one should add is as follows:

-   Log in as root or a user with side rights
-   Define type of the software archive (deb/deb-src). 
-   Define repository URL.
-   Define the distribution code name.
-   Define other repository components or categories (main, restricted, universe and multiverse). 
- **Usually  all of the above options are provided by software owner**
- In order to be able to install software from repository, one needs to import a public key from of the repository `curl -L [Key URL] | sudo apt-key add -`
- Run `sudo nano /etc/apt/sources.list` or `sudo nano /etc/apt/sources.list.d/filename.list` 
- In editor add line `[deb/deb-scr] [Repo URL] [System codename] [main] ` 
- **The line above is specified by software provider**
- Close the file
- `sudo apt update` To update information of packages present in the repository
- `sudo apt install [reponame]` To finally install the software

#### Worked example (MongoDB):

- ` curl -L https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -` 

- `sudo nano /etc/apt/sources.list.d/mongodb-org-5.0.list`

- In nano editor enter: `deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse` - all of it was specified by software provider

- `sudo apt-get update`

- `sudo apt-get install mongodb-org`

#### Removing:

- `sudo nano /etc/apt/sources.list` and remove the unwanted entry 
- (OR) `sudo rm /etc/apt/sources.list.d/filename.list`
- `sudo apt-key list` will list all keys, that apt trusts (has)
- Find the entry and copy its hex code
- `sudo apt-key del "[hex_keycode]"`
- `sudo apt update`

#### Worked example (MongoDB):

- `sudo rm /etc/apt/sources.list.d/mongodb-org-5.0.list`
- `sudo apt-key list` (MongoDB had F567 9A22 2C64 7C87 527C 2FBC B00A 0BD1 E2C6 3C11)
![image](https://github.com/v-bliznyukov/labsSNA/blob/main/list_keys.png)
- `sudo apt-key del  E2C63C11` - one can specify last 8 symbols of the key id
- `sudo apt-get update`


### 3.  When do you need to get public key for the usage of the remote repo? Provide an example.

APT that is used to update repository packages always checks signature of the release files. Moreover, it requires repositories to provide most recent key. This ensures that the provider can be trusted. Without this key, apt will refuse apt-get update. Therefore for every 3rd party repository key is needed in order for apt-secure to verify it. (All default installations already have a key)

#### Example: 
- As shown in previous example, for adding 3rd party MongoDB repository the public key is needed.

## Part 2

### 1.  Describe how “top” works. Explain all important fields in its output based on your system resources.
![image](https://github.com/v-bliznyukov/labsSNA/blob/main/top.png)
<code>top</code> command displays information about currently running processes/threads, system and resource usage. Moreover, <code>top</code> updates this information every time interval. User can specify the number of refreshments until quitting. 
There are several important fields of the output:
1. Process Id (PID)
2. User - who created a process
3. PR - priority of the task
4. NI - Nice Value, used to identify priority of a process. The lower the NI, the higher the priority. Default is +10.
5. VIRT - how much virtual memory is used by process
6. RES - how much of physical memory is used
7. SHR - how much shared memory is used by process
8. S - process status
10. %CPU - what percent of CPU is used
11. %MEM - what percent of memory is used
12. TIME+ - CPU time of a process (time it actually uses a CPU)
13. COMMAND - command/application that started the process

### 2.  Explain briefly each process states from the output of the “top” command.
There are several states process can be in:
1. *D* = uninterruptible sleep. Process waits for some event to happen to continue execution.
2. *I* = idle process i.e. has nothing to do.
3. *R* = running, executing right now
4. *S* = sleeping, process needs some resource, which is currently unavailable. Therefore, process stops execution for some time.
5. *T* = stopped process
6. *Z* = zombie. State that a process enters after exiting time and before parent releases it.
**Optionally all processes may have:**
7. *s* = leader of the session
8. *l* = threaded process
9. *N* = low priority. It is Nice to other processes)
10. *>* = high priority process
11. *+* = process is running in a foreground

For output of my system, the states are the following:
- S,Ss, Ssl, SNl, Sl+ (for tty)
- I<
- R+ (for ps aux command)
- T


### 3.  What happens to a child process that dies and has no parent process to wait for it and what’s bad about this?

When a child process terminates it releases the occupied resources except for the slot in process table (like PID). Then it sends a message to parent process, parent determines whether the exit was successful or not, end cleans the PT slot. In case when parent is dead also, there is no one to release process. Therefore, process stays in Zombie state and it cannot be killed since it does not exists. Apart from space usage, a lot of such processes occupy a lot of process IDs which may prevent other processes from running. 

### 4.  How to know which process ID used by application?
<code> ps aux | grep *application name* </code>

`ps aux` will return all processes (including not attached to terminal) and list the ones that have application name in the process table entry. After it, one needs to check what command is used by the process.
For example:
![image](https://github.com/v-bliznyukov/labsSNA/blob/main/find_name.png)
Here, we see the actuall running app is in process 15376, which is indicated by R state and md5sum /dev/sda command, while process with id 15375 is executing sudo command and is currently sleeping. This is because sudo is the parent process, it forks a child, which then does the execution. 


### 5.  What is a zombie process and what could be the cause of it? How to find and kill zombie process?

As discussed, process enters Zombie state between exiting and parent process releasing it (process has finished but still in the process table). A process may remain in Zombie state for a long time if parent process does not notice child's termination. This child process occupies some memory in process table, however, a lot of such entries may be a problem (preventing other processes from running). That many Zombie processes show that application has some troubles executing.
 
The command to find Zombie process can be:

<code> ps axo pid,stat | awk '$2 ~ /^Z/ { print $1 }' </code> 

However, this will consider that system had identified this process as Zombie and may not always be a reliable way. 

Note:
-   a = show processes for all users
    
-   x = also show processes not attached to a terminal

-   o = user specified format 

`ps axo pid,stat,ppid,comm | egrep "Z|defunct"`

The command above gets a list of all running processes and takes user specified fields; and egrep iterates over that list trying to find Z or defunct in the line. Defunct means that process has finished or terminated due to some error. Either way it is not functioning. 

Also, one can use top command to see the number of identified Zombie processes in a system.

Technically process is already dead, so it will not respond to any signals, so sending signals directly will not help. One can try several options:
- Identify parent process with `ps -o ppid= [Zombie Id] `
- Send a message that child has finished explicitly `kill -s SIGCHLD [Parent Id]`

OR
- Kill the parent process with `kill -9 [Parent PID]`
Terminating a parent process will trigger init process to take over the child processes. Then the init performs necessary deletion of Zombie processes.

OR (In extreme case)
- Restart the system
