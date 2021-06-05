# Linux General Usage



### Find files

Find *.txt

```text
find -name *.txt 
```

Case insensitive find

```text
find -iname *.txt 
```

Find things changed within 1 day

```text
find -mtime 1 
```

### Show size of a folder

```text
du -sh <folder>
```

### Archiving tools

```text
yum install atool
atool -x <file>
```

### Symbolic links

Create a symbolic link:

```text
ln -s (source) (link name)
```

A symbolic link is like a shortcut.  They can point to files, directories and remote locations.

### Shutdown, reboot

* shutdown -h now
* shutdown -r now

### Mounting using sshfs

Mount: 

```text
sshfs -o allow_other -o idmap=user ms@snowstorm:/ /mnt/snowstorm
```

Unmount:

```text
fusermount -u /mnt/snowstorm
```

### Mounting Windows shares

Mount:

```text
mount -t cifs '\\zen\c' /mnt/zen -o username=ms
```

Unmount:

```text
umount -l /mnt/zen
```


### X-Windows display host

Disable access control so other hosts can use your display

```text
xhost + 
```

Show your display number

```text
echo $DISPLAY 
```

Then on another host, send something to this display using the IP and display number.

```text
xeyes -display 192.168.0.7:0.0
```

(IP):(display number).(screen number)

### File permissions

Using letters.

* chmod (u/g/o/a) (+/-) (r/w/x/X/s/t)
    * u = user
    * g = group
    * o = other
    * a = all
    * r = read
    * w = write
    * x = execute
    * X = execute only if directory
    * s = run as the owning user/group
    * t = sticky bit - files cannot be deleted from a directory with this set.
* chmod u+rwx = user read, write, execute
* chmod g+rwx = group read, write, execute
* chmod o+rwx = other read, write, execute
* chmod a+rwx = all of the above read, write, execute

Using codes.

* chmod (user,group,other)
    * 7 = read, write, execute
    * 6 = read, write
    * 5 = read, execute
    * 4 = readonly
    * 3 = write, execute
    * 2 = write only
    * 1 = execute only
    * 0 = none
* chmod 700 = user read, write, execute; group and other none
* chmod 770 = user and group read, write, execute; other none
* chmod 777 = all read, write, execute

### Random commands

* ll = ls -l
* locate todo.txt = find todo.txt, searches every where (find searches the current directory only)
* ssh -t (user)@(host) (command) = run a command via ssh
* comm = useful tools to compare files and output matching/different lines from each file
* whatis = prints one liner description of a command
* cd - = change to previous directory
* !$ = reuse previous parameter (bash only)
* !! = reuse previous command (bash only)


