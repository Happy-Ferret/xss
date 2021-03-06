xss
===

[xss](http://github.com/9wm/xss) is a suite of X screensaver
utilities.  You can use shell scripts to glue the tools together to
build your own screen saver and/or locker.  You can use any
`xscreensaver` hack instead of the built-in `magic` hack, or you can use
`xlock` if you prefer.


The programs
------------

`xss` uses the decades-old MIT-SCREEN-SAVER extension to launch a
program when the X server turns on the built-in screen saver.  Unlike
`xautolock`, `xss` blocks until the X server says it's time to do
something.

`xsswin` makes a full-screen black window and runs some other program,
passing along the window ID in the environment (`$XSS_WINDOW`) and
possibly as an argument (`XSS_WINDOW` in argv is replaced with the ID).

`xkeygrab` grabs the keyboard and mouse, and echoes all typed lines to
stdout.

`xcursorpos` prints out the x and y coordinates of the cursor.

`xbell` sounds the X server's bell.

`magic` is a reimplementation of the "magic" screen saver from After
Dark.  It might look weird at 8bpp.


Examples
--------

Tell the X server to launch the screen saver after 90 seconds idle:

    $ xset s 90

Run like `xautolock`:

    $ xss xlock -mode qix &

Just run a screen saver, don't lock

    $ xss -w /usr/lib/xscreensaver/deco -window-id XSS_WINDOW &

Launch a program called "screenlock" when you're idle:

    $ xss screenlock &

An simple "screenlock" script:

    #! /bin/sh
    
    xsswin magic XSS_WINDOW & pid=$!
    xkeygrab | (while read l; do [ "$l" != "secret" ] && break; done)
    kill $pid

A more complex "screenlock" script which locks the screen awaiting a
pass phrase with the right md5 checksum.  After 4 seconds of being
locked, it pauses mpd (iff it was playing).  When the screen is
unlocked, mpd is resumed (iff it was playing beforehand).  The script
won't lock if the cursor's at the top of the screen.

    #! /bin/sh
    
    xcursorpos | (read x y; [ $y -lt 20 ]) && exit 0
    mpc | fgrep -q '[playing]' && playing=1
    xsswin magic XSS_WINDOW 2>/dev/null & xsswin=$!
    (sleep 4; [ $playing ] && kill -0 $xsswin 2>/dev/null && mpc --no-status pause) &
    xkeygrab | (
        while read l; do
            md5s=$(echo -n $l | md5sum | cut -d\  -f1)
            if [ $md5s = 'a37c87558d98e9fe0484e09070268be1' ]; then
                break
            fi
            xbell
        done
        )
    kill $xsswin
    [ $playing ] && mpc --no-status play


History
-------

AIX apparently had something also called `xss` which did almost exactly
what mine does, but with command-line options.  `magic` is similar to
the `qix` hack from xscreensaver and xlock.  I'm not aware of anything
else like the rest of the programs, which is why I wrote them.

I lifted some code from `beforelight` from the X11 distribution, and
from `slock` from [suckless.org](http://suckless.org/).  Both have a
BSD/X11-like license.


------

Neale Pickett <neale@woozle.org>
