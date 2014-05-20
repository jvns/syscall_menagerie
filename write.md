# write

`write` writes to files. Unlike `open`, it's not the *only* way to
write to a file: you could also `mmap` the file read-write and change
the memory. But it's a common way, and very worth knowing about.

When you run


```
f = open('file.txt', 'w')
f.write("hey!")
```

you're using `write`.

Here's my favorite way to use `write` to solve problems:

## Find out what file a process is logging to

I have a web server on my laptop (`lighttpd`), and I don't know where
it logs its errors. I could read its configuration, or Google "where
does lighttpd log", or look in `/var/log`.

Since `lighttpd` is a running process, we'll need to look up its PID
first. I find the PID with `ps aux | grep lighttpd`. It's 16141.

Then we can see where it's writing to! I start stracing --

```
$ sudo strace -p 16141 -e write
```

and reload a webpage.


