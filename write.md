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
$ sudo strace -p 16141 -s 3000 -e write
# the -s 3000 is to display the first 3000 characters of each string
```

and reload a webpage in my browser. Here's what we see!
```
write(3, "2014-05-20 18:41:36: (mod_fastcgi.c.2701) FastCGI-stderr:
PHP Notice: Undefined index: q in /home/bork/www/index.php on line 6\n
\n", 130) = 130
```

Okay, neat! But what's `3`? That's a file descriptor. If I do
`ls -l /proc/16141/fd/3`, I see:

```
$ sudo ls -l /proc/16141/fd/3
l-wx------ 1 root root 64 May 20 19:26 /proc/16141/fd/3 -> /var/log/lighttpd/error.log
```

That's not a huge surprise -- I probably could have guessed that it
was in `/var/log/lighttpd`. But this way was foolproof :)
