---
layout: post
title: "Writing a Linux Keylogger"
date: 2016-04-16
---

So I had the brilliant idea of writing a keylogger for Linux. In Linux, there is
the `/dev/` folder that contains a lot of FIFO pipes for certain processes.
Among these, there were 2 that stood out to me regarding the keyboard;

1. `hidraw0`
2. `hidraw1`

**HID** means **H**uman **I**nterface **D**evice. These are peripherals, like
keyboards, mice, touchpads, etc.

Doing a simple `ls -l` yielded that these were owned by `root`, so I used `sudo
cat /dev/hidraw0` to check the output. To my great satisfaction, as soon as I
typed in some letters, it spit out a bunch of gibberish and special symbols: low
decimal value keycodes that were converted to ASCII.

To check out what these were, I opened a Python shell as `root`:

<div class="codeblock">
<pre>
$ sudo python
>>> f = open("/dev/hidraw0")
>>> while True:
        print ord(f.read(1))
</pre>
</div>

What this yielded was many many printouts of a bunch of numbers. Through some
experimentation, I found that every keystroke was a list of 10 bytes:

<div class="codeblock">
<pre>
$ sudo python
>>> f = open("/dev/hidraw0")
>>> while True:
        print [ ord(x) for x in f.read(10) ]
</pre>
</div>

I don't know if it's 10 for all keyboards, but I'm using a MacBook Air 4.2, running
LXLE. After it started printing out, I noticed a few things:

The 2nd element (index 1) of the list correlated to modifier keys. In fact, they
behaved in the way that permissions are stored in C: bit switches. Through some
experimentation, I discovered the following schema:

<div class="codeblock">
<pre>
MODIFIER KEYS: 8 BIT CODE:

0 0 0 0 0 0 0 0
| | | | | | | |___ Left Control
| | | | | | |_____ Left Shift
| | | | | |_______ Left Alt
| | | | |_________ Left Super
| | | |___________ Right Control
| | |_____________ Right Shift
| |_______________ Right Alt
|_________________ Right Super
</pre>
</div>

I was unsure about the keycodes themselves, and since I was a bit bored in
class, I decided to test out every key that was available on the built-in
MacBook Air 2011 keyboard.

The source can be found here: [keylogger](https://github.com/elc1798/keylogger)

