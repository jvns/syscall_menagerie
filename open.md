# `open` 

`open` opens files. It was the first system call I learned, and I
reach for it a lot.

Whenever you run

```
f = open('file.txt', 'r') # python
int fd = open('file.txt', O_RDONLY) # C
```

or the corresponding code in your favorite programming language, it
always ultimately calls the `open` system call. Here are a few
scenarios where you might want to use `open`:

## Finding configuration files

Have you ever wanted to know which configuration files a program is
using, but don't feel like looking it up? You don't need to! It can't
read them without calling `open`.

Let's try this out on `lighttpd`, a tiny web server I have installed.

```
sudo strace -o /tmp/lighttpd_open -f -etrace=open /etc/init.d/lighttpd restart
```

After some digging (it opens a lot of files!), I find

```
16133 open("/etc/lighttpd/conf-enabled/90-javascript-alias.conf", O_RDONLY) = 3
16133 open("/etc/lighttpd/lighttpd.conf", O_RDONLY) = 3
```

I also found some neat files that I didn't know about before like
`/etc/protocols` and `/etc/services`, with huge lists of port numbers.

## Debugging shared library versions

A while ago I had a problem where I had the wrong version of a library
installed. I tried to fix it, but it wasn't working! I fiddled with my
`LD_LIBRARY_PATH`, `open` came to the rescue though!

Any time you start a program, it needs to `open`
all the libraries that it loads. For example, if I run

`strace ls 2>&1 | grep .so | grep -v ENOENT`

I get

```
open("/lib/x86_64-linux-gnu/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/librt.so.1", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/libacl.so.1", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
open("/lib/x86_64-linux-gnu/libattr.so.1", O_RDONLY|O_CLOEXEC) = 3
```


There are some familiar suspects: pretty much every program uses
`libc`, and `libpthread` is pretty important.

`open` let me find out exactly which library my program was opening
(surprise: the wrong one!) and fix the program (in this case by
deleting the offending library)

## Man page & source code

Man page: TODO

Source for open in the 3.5 kernel: TODO
