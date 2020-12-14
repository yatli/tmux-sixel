> sixel-tmux : a terminal multiplexer that properly display graphics because it does not eat escape sequences

### What is sixel-tmux

sixel-tmux is a fork of tmux, with just one goal: having the most reliable support of graphics.

In a console, this means:
 1. ANSI blocks and box-drawing characters <https://en.wikipedia.org/wiki/Box-drawing_character> to display ASCII and ANSI art <https://en.wikipedia.org/wiki/ANSI_art>
 2. ANSI escape codes, with a focus on SGR parameters <https://en.wikipedia.org/wiki/ANSI_escape_code#SGR_parameters> to render text attributes
 3. and the best of all: sixel graphics <https://en.wikipedia.org/wiki/Sixel> to display graphs <https://github.com/csdvrx/sixel-gnuplot>

This seems very simple requirements, yet a regular tmux fails on all 3:
 1. ANSI art is corrupted,
 2. some SGR parameters are supported, but the more exotic features like overline do not work, and some sequences may be intercepted and improperly changed
 3. sixels are not supported.

For a proper support of all these features, sixel-tmux is necessary but
not sufficient: the terminal must be able to decode and render the sequences
passed by sixel-tmux.

Therefore, a separate test suite <https://github.com/csdvrx/sixel-testsuite>
is provided to first check if a terminal can achieve these goals.

If you are dissatisfied by your current terminal, please consider mlterm
<http://mlterm.sourceforge.net/> or mintty <https://mintty.github.io/> that are
tested and known to work well.

Microsoft Window Terminal is very promising, but does not support sixels yet.
Please upvote the feature request <https://github.com/microsoft/Terminal/issues/448>

You can find a longer list of alternative terminals and terminal multiplexers through
related works such as <https://github.com/gizak/termui/pull/233>

## Usage

First, use to test-suite to see what your terminal supports:

For the colors:
1. start by 16-colors.sh
2. then try 256-colors.pl
3. see if you are lucky and can use a 24 bits palette with 24-bit-color.sh
4. if it works, try to switch the 16 colors palette between Solarized dark and light with toggle-solarized-dark.sh and toggle-solarized-light.sh

In the final test, after displaying the 4 continous color lines without bands
or sudden breaks, your screen should switch to a black background then back to
a clear yellowish background.

For the fonts:
1. start with `cat font-ansi-blocks.txt` to see block drawing characters and a test bear
2. run `font-vt100.sh` to see the basic box drawing characters
2. `cat font-ansi-box.txt` to see the full set of box drawing characters
3. `cat font-dec.txt` to check DEC VT220 extra characters
4. `cat font-test-all.txt` to check unicode support and overall support of the drawing characters

In the final test, boxes should be properly aligned, and lines should intersect cleanly.

If the font test fails, you may need to change your font as it may not support every character.

Alternatively, when using mlterm you can tell it to extend your preferred font with one or more other fonts, for example:
        ISO8859_1 = Iosevka SS04 18
        ISO8859_15 = Iosevka SS04 18
        ISO10646_UCS2_1 =Iosevka SS04 18
        ISO10646_UCS2_1_BOLD = Iosevka SS04 18
        U+2500-25ff=Segoe UI Symbol
        U+25C6 = Tera Special # Diamond                  ◆
        U+2409 = Tera Special # Horizontal tab           ␉
        U+240C = Tera Special # Form feed                ␌
        U+240D = Tera Special # Carrier return           ␍
        U+240A = Tera Special # Linefeed                 ␊
        U+2424 = Tera Special # Newline                  ␤
        U+240B = Tera Special # Vertical tab             ␋
        U+23BA = Tera Special # Horizontal line 1        ⎺
        U+23BB = Tera Special # Horizontal line  3       ⎻
        U+2500 = Tera Special # Horizontal line  5       ─
        U+23BC = Tera Special # Horizontal line  7       ⎼
        U+23BD = Tera Special # Horizontal line  9       ⎽

When everything works to your satisfaction, install libevent2 and ncurses, then
compile with: `./configure && make && make -j8 install`

Then:
1. install sixel-tmux.terminfo: `tic sixel-tmux.terminfo; tic -o ~/.terminfo sixel-tmux.terminfo`
2. start sixel-tmux: `sixel-tmux`
3. select sixel-tmux: `export TERM=sixel-tmux`
4. verify sixel-tmux is used: `tput smglr|base64` should return GzcbWz82OWgbWyVpJXAxJWQ7JXAyJWRzGzg=

You can then run the test-suite <https://github.com/csdvrx/sixel-testsuite> to check if what was working in your terminal before sixel-tmux is still working.

If it isn't, oops: it means sixel-tmux broke something. Patches are welcome!

### Current status and tests failed

Sixels fully work, but images are not redrawn when returning from another buffer (snake.six)

Overlines are not working yet (ansi-vte52.sh)

Ansi blocks are separated until you move to another buffer and back (font-ansi-blocks.txt)

24-bit mode is not supported (24-bit-color.sh)

dynamic switching of the 16 colors palette is not working (toggle-solarized-dark.sh/toggle-solarized-light.sh)

vertical split does not plays well with ansi and sixel

### Example inside mintty

Here is what my terminal looks like while I am editing this file in VIM

![mintty running tmux split window and displaying sixels inside](https://raw.githubusercontent.com/csdvrx/sixel-tmux/master/sixel-tmux.jpg)

### Why a fork

To support slow terminal with little bandwith, tmux started implementing
some controversial features after version 2.3. For example, when a lot of text
arrives, tmux start silently discarding the output until a redraw can be done.

Bug reports <https://github.com/tmux/tmux/issues/1019#issuecomment-318284445>
<https://github.com/tmux/tmux/issues/1502#issuecomment-429710887> mention
serious issues, such as being unable to cat a long text file without some parts
being cut off, so this questionable feature for slow terminals impacts many usecases. 

Among other things, it breaks sixel support. Even when not using sixels, it
seems very wrong for tmux to be taking the initiative of silently discarding
parts of the outputs, without providing any command line flags to override that
"feature".

As it can't be disabled, this anti-feature is not just a questionable default,
but a perverse feature: I believe breaking the integrity of the output of
whatever command was run, with no recourse, is not acceptable for a terminal multiplexer.

Also, refusing to even provide a command-line toggle to opt-out is not
respectful of the end user freedom: no one should have to track patches to
allow basic functionality like properly displaying long text files!

The maintainer explicitely said several times that even if a patch was provided
to support sixels, the functionality would never be added to tmux, and it would
have to be a fork
<https://github.com/tmux/tmux/issues/44#issuecomment-119755304>

By becoming a permanent fork of tmux, sixel-tmux will do just what was asked.

sixel-tmux is not just made for sixels but for all graphical things that can no
longer be properly displayed inside tmux, including long text files.

Unfortunately, a permanent fork seems necessary as the problem seems to be more
general, and spreading: tmux intercept and improperly rewrites various escapes
codes, breaking other things
<https://github.com/tmux/tmux/issues/1391#issuecomment-403267557>

At this point, it is very unlikely that tmux will be fixed: all these
questionable choices start piling up, and make the correct display of what is
not just simple plain text inside tmux harder and harder.

This is a sad situation as tmux was one of the best terminal multiplexer, and
supported what GNU screen couldn't when too many questionable choices had been
accumulated and frozen in over 32 years of spaghetti code.

Therefore, all patches adding functionality to sixel-tmux are welcome-
including cleanups, and ports of tmux features. Even the features that may affect the
proper display of graphics and text are welcome, as long as they are off by
default since obscure usecases like limited bandwidth are now as rare as
serial ports.

### History

Hayaki Saito <https://github.com/saitoha/tmux-SIXEL> wrote a patch and released
a tmux-sixel fork in 2015.  As tmux keep adding features, Chris Steinbach
<https://github.com/ChrisSteinbach/tmux-sixel> tried to integrate them in
tmux-sixel.  Patches were released by others to add to the version of tmux
officialy packaged by distributions to have tmux-sixel features, like Mehdi
Abaakouk <https://gist.github.com/sileht/8759089d5620da763bb456a0ae82e4d0>

Yatao Li <https://github.com/yatli/tmux> had a branch that was the closet to upstream
tmux, but the most recent changes dated from 2017 and some sixel isues remained.

When I tried them, none had full support for ANSI SGR parameters. In 2019, as
there was no recent activity on any of these, I forked the most recent branch,
fixed the remaining issues to have good sixel support at least in the current
buffer, added the missing SGR parameters I needed, and fixed issues brough by
the inclusion of unavoidable anti-features in mainline tmux.
