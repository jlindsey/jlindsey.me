---
title: "Moving from OS X to Linux"
date: 2014/04/26
tags: Linux, OS X, Vim
---

For more or less my entire career as a web dev (and then DevOps) I've been
using OS X. Its combination of 'nix roots via BSD and a developer community
[making][1] [really][2] [stellar][3] [tools][4] make it a great choice for my
line of work.

Unfortunately, my MacBook Pro retina was starting to have some hardware issues
and I found myself needing a new primary laptop. I thought about getting
another MBP, but I'd been giving some thought to moving to Linux for a while
now. It made sense: being Ops, I spent the great majority of my time in a
fullscreen terminal anyway, SSH'd into various headless Linux servers; I had
been [using vim][5] as my primary text editor for some time; and I simply
wanted a change of scenery. So I took the plunge.

### My Goals

It's all great talking about making a change in my working environment, but if
it impacted my performance that would obviously not be good. So with that in
mind, I had a few goals before I set out:

* Try to use the keyboard as much as possible and the mouse as little as
possible.
* Get a better working knowledge of Linux.
* Be at least as productive as I was on a Mac.

### The Hardware

My requirements were simple: lightweight, good screen with a high native res,
i5 or better, at least 8GB of memory, and, of course, good Linux support. I
ended up going with the [Lenovo X1 Carbon][6] non-touchscreen version. I
upgraded it with 8GB (the maximum available) RAM and the i7 processor. It comes
standard with a 2560x1440 IPS screen. ThinkPads in general have always had a
great reputation for being more-or-less compatible with Linux out of the box
with little hacking required. The X1 is (mostly) no exception to that rule.

The keyboard is [a little odd][7], with the tilde key in the lower right,
home/end keys replacing capslock, and backspace/delete on another rocker key in
the upper right. And the most obvious: the function row being replaced by the
"adaptive keyboard", an OLED touchscreen (more on that in a bit). The only
thing that really trips me up is the tilde key; my muscle memory is so trained
to reach up-left for it. But I normally have my laptop on a stand with an
external keyboard and monitor anyway, so it's not a big deal to me.

The adptive keyboard is designed to have multiple "pages" that you select via
the Fn softkey. In Linux, this doesn't work. Thankfully, the first page
contains the actual function keys (which do work) so the only annoyance here is
that I'm unable to change the screen brightness or volume via the keyboard.
I've found workarounds for this, but they're not as handy has just being able
to reach up and press a key. The reason this doesn't work is kernel support,
and there is a fix in the 3.15 kernel. My OS is only on 3.13 though, and
upgrading completely broke the video drivers. Maybe someone else will have
better luck.

The touchpad is a "clickpad", which I guess is what we're calling MacBook-esque
whole-pad-is-a-button things now. It's extremely sensative and I had to
override some Synaptics settings for it so it was usable at all, and so it
wouldn't constantly select things while I was typing. The feel is a little
"mushy" and it doesn't have that satisfying *click* like the Apple trackpads
do, but as I'm trying to ween myself from the mouse in general I don't care too
much. There's also the signature ThinkPad trackpoint nipple in the middle of
the keyboard that I disabled in the BIOS because I hate those things.

Immediate problems that presented themselves after I first installed the OS
were relatively minor. Shutting the screen would put the machine to sleep but
it wouldn't wake after opening it again (easily fixed with a BIOS update). As I
mentioned, the trackpad was over-sensative. And the fingerprint scanner isn't
supported yet by `fingerprint-gui`.

### The Software

[![Screenshot][9]{: class="img-thumbnail"}][9]{: data-lightbox="1"
data-title="Workspace with two terminals"} Ubuntu 14.04 just realeased so I
decided to go with that. It's really hard to beat its stellar out-of-the-box
hardware support and package repositories.  In fact, I ended up going with
[Xubuntu][8] as I wanted a more lightweight distro on which to install a tiling
WM. To the left is a screenshot of more-or-less the finished product, running
[i3][10] as the WM; two terminals tiled horizontally running Vim and the jekyll
server with this post open; and [i3pystatus][11] as the i3 menubar replacement.

Running i3 within Xfce provides a few advantages: I have access to Xfce's GUI
settings managers if I need them; and I can take advantage of its built-in
screen DPI settings so I can run at the full native resolution of my screen
with scaling, just like how OS X handles retina displays.

Xmonad seems to be the hotness at the moment (especially among Arch users) but
after trying it for a while I ended up going with i3 for a number of reasons.
For one, I don't speak Haskel. It's a shortcoming and outs me as totally square
and uncool, I know. But I really didn't want to have to learn an entire
functional language just to configure my WM. For another, i3 was built [with
multi-display support specifically][11], and it's dead simple to configure with
almost no work.

i3pystatus is quite a nice replacement for i3's built-in i3bar written in
Python 3. It comes with a great assortment of plugins such as battery life,
wifi, and system load. It's also very extensible. As I mentioned above, I'm not
able to change my screen brightness because of the OLED keyboard bug so [I
wrote a plugin][13] for the bar to display the brightness and raise/lower it
with clicks.

[![Screenshot][14]{: class="img-thumbnail"}][14]{: data-lightbox="2"
data-title="Workspace with Vimperator"} For browsing, I've used Chrome for a
long time. I thought since I'm giving different things a try, let's see how
Firefox is doing these days. I installed [Vimperator][15] to further the "less
mouse, more keyboard" goal and after a day or two of getting my bearings, I
love it. It feels natural, it's *extremely* configurable, and keeping my
fingers on the home keys as much as possible is awesome. Also installed are
Firebug (of course), Ghostery, and Tree Style Tab.

Issues with the software part of things were almost completely confined to
Skype, that bane of computer users everywhere. I ended up making a [wrapper
script][16] placed ahead of the Skype binary in my `$PATH` to fix issues with
shared lib loading and PulseAudio4.

### The Takeaway

Moving from one OS to another is a big leap, but I'm glad I made it. I'm
learning a ton about Linux, about tweaking settings and setting kernel boot
flags. I'm getting more and more comfortable with a keyboard-only setup, and
I've definitely noticed some speedups in my workflow from keeping my hands on
the home row. I still have my MBP for compiling and testing OS X and iOS apps
on, but I mostly bring it to work because my iTunes library is on it.


[1]: http://www.iterm2.com
[2]: http://atom.io
[3]: http://brew.sh
[4]: http://pow.cx
[5]: https://github.com/jlindsey/vim
[6]: http://shopap.lenovo.com/au/en/laptops/thinkpad/x-series/x1-carbon/
[7]: http://cdn.arstechnica.net/wp-content/uploads/2014/01/thinkpad-x1-carbon-keyboard.jpg
[8]: http://xubuntu.org
[9]: /images/moving-from-os-x-to-linux/workspace-with-two-terminals.png
[10]: http://i3wm.org
[11]: https://github.com/enkore/i3pystatus
[12]: http://i3wm.org/docs/userguide.html#multi_monitor
[13]: https://gist.github.com/jlindsey/11359017
[14]: /images/moving-from-os-x-to-linux/workspace-with-vimperator.png
[15]: http://www.vimperator.org/vimperator
[16]: https://gist.github.com/jlindsey/11359155
