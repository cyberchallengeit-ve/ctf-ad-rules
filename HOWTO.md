HOW-TO CTF
==========
The aim of this document is to provide some hints that could be useful while playing an Attack/Defence CTF. Remember that each team has to manage its own virtual machine, running some vulnerable services, hence you may need to know some useful commands to list or kill processes currently in execution, find out active connections on your machine, connect to a MySQL database via command line and so on. Furthermore, during the CTF you should develop some scripts to avoid to perform manual operations at each round, try to automate repetitive tasks as much as possible!

Clearly this cannot be a comprehensive guide: if you have suggestions please tell us and we will improve this document!

### Change your password
The first thing you must do when the competition starts is to change the `root` password of your virtual machine: since you will connect directly as `root` with `ssh`, it is enough to use the command `passwd`.

```Bash
root@team:~# passwd
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```

Choose a complex and long password here. Now, instead of typing it every time you connect to your VM, it's much better to set up SSH public-key authentication to connect to the vulnbox. We strongly advise all teams to prepare a file with the SSH public-keys of all team members and add them to `/root/.ssh/authorized_keys` as soon as the competition starts.

### Switch user
In your machine there could be a different user for each service: since it is a good habit to use `root` only when strictly necessary, you may consider the possibility of switching to the user that owns the service on which you want to work. This can be done with the command `su - <user>` if you want to go back to the previous user, you can use the combination `CTRL + D` or the command `exit`.

```Bash
root@team:~# su - pin
pin@team:~$ id
uid=1001(pin) gid=1001(pin) groups=1001(pin)
pin@team:~$ exit
logout
root@team:~# 
```

### The manual
If you need to know more information about a command, you must simply type `man <command_name>` on the command line. You can search for a word typing `/<word>` once the manual page is open and, if you press `n`, the next occurrence will be shown. You can move inside the page using arrow keys. When you have finished, type `q` to exit.

You can also get the documentation of many functions of the standard C library using `man 3 <function_name>`.

Do you want to know more about `man`? Read the manual of the manual :)

### Files search
`find` is a very powerful tool for searching files inside a given path with some particular properties: for instance you can ask to list files that belong to a given user, search for files on which you have particular permissions and so on. You can also combine different properties using logical operators.

For instance, this command allows you to find regular files (not directories) owned by the specified user that are readable by everybody:

```Bash
diff@team:~$ find /home/pin/ -user pin -perm /a+r -type f
/home/pin/challenge/pin.py
/home/pin/challenge/server.py
/home/pin/challenge/pin.pyc
```

On the other hand, if you want to search, only in the working directory, for files whose names start with `.b` or with permissions exactly `755`, we can use:

```Bash
diff@team:~$ find . -maxdepth 1 -name '\.b*' -o -perm 755
.
./.bash_history
./.bash_logout
./.bashrc
```

Refer to the manual if you want to know more about this utility.

### Process management
A common way to obtain complete information about the processes currently in execution on the system consists on using `ps` with options `aux`. Here is an example of (part of) the result produced by the command:

```diff@team:~$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.3   3504  1764 ?        Ss   Feb11   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Feb11   0:01 [kthreadd]
diff       238  0.0  1.6  14808  8444 ?        S    Feb11   0:01 /usr/bin/python /home/diff/challenge/server.py
diff     17417  0.1  0.2   4936  1232 pts/2    T    18:59   0:00 nano
diff     17418  0.0  0.2   5204  1192 pts/2    R+   18:59   0:00 ps aux
diff     32098  0.0  1.2  12400  6256 pts/4    S+   Apr17   7:44 python -m SimpleHTTPServer
```

The fields you may be interested in are probably `USER`, `PID` and `COMMAND` that respectively denote, for each process, the user that started it, its process identifier and the command given to start it.

You can kill a process using the command `kill` and specifying the PID: for instance, if we want to kill `nano`:

```Bash
diff@team:~$ kill -9 17417
diff@team:~$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.3   3504  1764 ?        Ss   Feb11   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Feb11   0:01 [kthreadd]
diff       238  0.0  1.6  14808  8444 ?        S    Feb11   0:01 /usr/bin/python /home/diff/challenge/server.py
diff     17425  0.0  0.2   5204  1192 pts/2    R+   19:00   0:00 ps aux
diff     32098  0.0  1.2  12400  6256 pts/4    S+   Apr17   7:44 python -m SimpleHTTPServer
```

If you want to kill processes that are not owned by your current user, you have to switch to the one that owns the process (or to `root`).

You can even specify to kill processes with a (partially) given name or that belong to some specified users and so on using `pkil`: refer to the manual for more information.

### Network connections
Information about active network connections can be shown using `netstat` as follows:

```Bash
diff@team:~$ netstat -natup
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      32098/python    
tcp        0      0 0.0.0.0:42765           0.0.0.0:*               LISTEN      238/python      
tcp        0      0 0.0.0.0:42766           0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp        0    672 192.168.69.22:22        192.168.69.159:51987    ESTABLISHED -               
tcp6       0      0 :::22                   :::*                    LISTEN      -  
```

Column `Proto` reports the transport protocol in use, `Local Address` and `Foreign Address` denote, respectively, addresses of the parts involved in the connection, `State` is the state of the connection (except for connectionless protocols) and `PID/Program name` shows the PID and the name of the program that has created the connection. The last column shows information only of processes owned by the user that started the command: if you want to see them all, you have to run the command as `root`.

For instance, the first row says that there is a Python program listening on the port 8000, while the fifth says that there is a connection on the interface with IP address 192.168.69.22 on the port 22 with a machine with address 192.168.69.159 on the port 51987.

You can quickly open a connection with an host using `nc` (netcat), specifying the IP address (or the domain name) and the port. Moreover, you can use netcat to listen for a connection, check the manual!

Furthermore, it's pivotal during the CTF to analyze the traffic that goes through your VM. To do so, you may find `tcpdump` and `wireshark` of great help. If you're good at traffic analysis you may even be able to _steal_ attacks performed by other teams against your VM!

### Remote file copy
If you want to copy some files from your virtual machine to the computer you're using or viceversa, `scp` is what you need. Another handy tool that can be used to synchronize a remote and a local directory is `rsync`. Additionally, if you find uncomfortable working with a text editor directly on the VM, you can consider mounting a remote directory on your laptop using `sshfs` and patch services with your editor of choice. 

### Command Line Editors
Copying a file to your computer each time you have to modify it and then moving it back to the virtual machine is not only boring: you also have to pay attention that the attributes of the copied file, such as user and group owner or permissions, are the same of the previous version. Using a command line editor is not a bad idea at all: one of the simplest editors available is `nano`. Of course, there are also more advanced command line text editors, such as `vim`.

### Tcpserver
In CTFs it's possible to see `tcpserver` in action. It allows the use of C-compiled programs by remote users without having to deal manually with sockets: when a connection is received on the port associated to a certain service, `tcpserver` runs the program with file descriptors `0` and `1` (that denote standard input and standard output) reading from and writing to the network. Suppose you have the following program:

```C
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
	char s[100];

	printf("What's your name? ");
	fflush(stdout);
	scanf("%99s", s);
	printf("Hi %s, how are you?\n", s);

	return EXIT_SUCCESS;
}
```

You can bind the service to a port on a specific address typing `tcpserver <options> <ip_address> <port> <absolute_path_to_executable>`. For instance:

```Bash
$ tcpserver -H 0 12345 ~/name
```

Now you can use netcat, for example, to interact with the service:

```Bash
$ nc localhost 12345
What's your name? l33thacker
Hi l33thacker, how are you?
```

Actually, if you're playing a CTF where there is a service relying on `tcpserver`, you shouldn't need to run the command by yourself: when you fix a vulnerability in a C program, quite often by patching the binary since most of the time no sources are provided, it is enough to replace the executable of the service with the new one to use it.

### Interaction with DBMSs
During the competition you may need to interact with the MySQL DBMS installed on your virtual machine for different reasons: for instance, to find out where are located the flags of services that rely on a database for data storage, in order to plan your attack.

You can open a simple SQL shell using the command `mysql -u <user> -p`, where `user` is the name of a user registered in the DBMS.

Here are some useful commands: to see databases available on the system, you can use `show databases`:

```SQL
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)
```

If you want to see information about tables of a certain database or columns of a particular table, you can proceed as follows.

```SQL
mysql> show tables from test;
+----------------+
| Tables_in_test |
+----------------+
| teams          |
| users          |
+----------------+
2 rows in set (0.00 sec)

mysql> show columns from test.users;
+----------+------------------+------+-----+---------+----------------+
| Field    | Type             | Null | Key | Default | Extra          |
+----------+------------------+------+-----+---------+----------------+
| id_user  | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name     | varchar(30)      | NO   |     | NULL    |                |
| surname  | varchar(50)      | NO   |     | NULL    |                |
| nick     | varchar(20)      | NO   | UNI | NULL    |                |
| password | varchar(32)      | NO   |     | NULL    |                |
| id_team  | int(10) unsigned | NO   | MUL | NULL    |                |
+----------+------------------+------+-----+---------+----------------+
6 rows in set (0.00 sec)
```

If you want to use a particular database, so that each time you don't have to specify the DB name before the table on which you want to operate, give the command `use <database>`. Of course you can also execute standard SQL queries to retrieve, insert, update or delete data.

Other popular choices among DBMSs are [SQLite](https://www.sqlite.org/index.html) and [PostgreSQL](https://www.postgresql.org/). Refer to their websites for the documentation.

### HTTP requests with Python
When you find a flag, you have to submit it on a login protected site to earn points and rise in ranking: doing it manually for all the flags is really annoying, since you will have to collect as much flags as you can during the different rounds in which the competition is divided.

Manual submission is ok for a couple of ticks while testing your exploit, but remember that you are expected to write your own scripts to perform attacks automatically at each round against all your opponents. You can easily perform HTTP requests using a Python module that is installed on your virtual machines, <a href="http://docs.python-requests.org/en/latest/">Requests</a>.