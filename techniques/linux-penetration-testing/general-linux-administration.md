---
description: A primer on administrating your linux attach machine
---

# General Linux Administration

## Manipulating Files

### Read a file

```bash
cat [filename.txt]
head [filename.txt]
tail [filename.txt]
hexdump -C [filename.txt]
tac [filename.txt]
```

### Edit a file 

```bash
nano filename.txt
vi filename.txt
```

### Move a file 

```bash
cp file/path/name.txt new/file/path/
```

## Managing Users

#### Create a User

```text
sudo useradd [username]
```

####  Set a users password

```text
passwd [username] 
```

## The Apache Webserver

Especially when you are competing in Boot 2 Root or local network competitions, running an apache webserver comes in handy. From transferring tools to your target with commands like wget and curl, to stealing authentication tokens and appending them to apache logs, knowing how to administer a simple webserver is key. 

#### Checking if Apache is installed 

```text
dpkg --get-selections | grep apache
```

#### Starting the  Server

There are several ways to start the server, each line below is a different method.

```text
sudo service apache2 start
sudo systemctl start apache2
```

#### Stopping the Server

```text
sudo service apache2 stop
sudo systemctl stop apache2
```



