# contrast

A program to check WCAG contrast rules for foreground/background colors.

I've had this in my home toolbox for years, but recently needed this on another
machine, so I'm making it available so I can find it again.

Updated in 2026 to optionally try to find the smallest perceptual change in
foreground or background lightness (or both, or also hue, see options!) to make
the contrast pass WCAG AA or AAA, for large or normal text, depending on the
options passed: I had a need to see by how much a color that passes AA for small
or large text already needed to change in order to also pass AAA for small or
large text, such that "just a bit more change" might be enough to meet the
higher, but better, standard.

## Usage

At its simplest, you can pass a FOREGROUND and a BACKGROUND color, in hex
notation, and it'll output information about the contrast for the two:

```bash
$ ./contrast ff0000 111112
fg ff0000 hsl(0°, 100%, 50%)
on 111112 hsl(240°, 2%, 6%)
 = EXAMPLE
 =  4.720:1 contrast
✔PASS AA  large
✘FAIL AAA large
✘FAIL AA  text
✘FAIL AAA text
```

The "EXAMPLE" text is suitably ANSI colored, using `\e[38;2;R;G;Bm` for the
foreground color, and `\e[48;2;R;G;Bm` for the background color.

If given the `-l` option (which must come after the FOREGROUND and BACKGROUND
values), it outputs all the information in one line:

```bash
$ ./contrast ff0000 111112 -l
fg ff0000 hsl(0°, 100%, 50%)   on 111112 hsl(240°, 2%, 6%)    = EXAMPLE =  4.720:1 contrast ✔PASS AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
```

This makes it easy/possible to use it for gauging which colors of a whole
palette can be used with each other without failing any WCAG specification,
i.e.:

```bash
$ export PALETTE='ff0000 eeeeee 000000 111111'
$ for fg in $PALETTE; do for bg in $PALETTE; do ./contrast "$fg" "$bg" -l ; done ; done
fg ff0000 hsl(0°, 100%, 50%)   on ff0000 hsl(0°, 100%, 50%)   = EXAMPLE =  1.000:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg ff0000 hsl(0°, 100%, 50%)   on eeeeee hsl(0°, 0%, 93%)     = EXAMPLE =  3.446:1 contrast ✔PASS AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg ff0000 hsl(0°, 100%, 50%)   on 000000 hsl(0°, 0%, 0%)      = EXAMPLE =  5.252:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✘FAIL AAA text
fg ff0000 hsl(0°, 100%, 50%)   on 111111 hsl(0°, 0%, 6%)      = EXAMPLE =  4.723:1 contrast ✔PASS AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg eeeeee hsl(0°, 0%, 93%)     on ff0000 hsl(0°, 100%, 50%)   = EXAMPLE =  3.446:1 contrast ✔PASS AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg eeeeee hsl(0°, 0%, 93%)     on eeeeee hsl(0°, 0%, 93%)     = EXAMPLE =  1.000:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg eeeeee hsl(0°, 0%, 93%)     on 000000 hsl(0°, 0%, 0%)      = EXAMPLE = 18.100:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg eeeeee hsl(0°, 0%, 93%)     on 111111 hsl(0°, 0%, 6%)      = EXAMPLE = 16.275:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg 000000 hsl(0°, 0%, 0%)      on ff0000 hsl(0°, 100%, 50%)   = EXAMPLE =  5.252:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✘FAIL AAA text
fg 000000 hsl(0°, 0%, 0%)      on eeeeee hsl(0°, 0%, 93%)     = EXAMPLE = 18.100:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg 000000 hsl(0°, 0%, 0%)      on 000000 hsl(0°, 0%, 0%)      = EXAMPLE =  1.000:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg 000000 hsl(0°, 0%, 0%)      on 111111 hsl(0°, 0%, 6%)      = EXAMPLE =  1.112:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg 111111 hsl(0°, 0%, 6%)      on ff0000 hsl(0°, 100%, 50%)   = EXAMPLE =  4.723:1 contrast ✔PASS AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg 111111 hsl(0°, 0%, 6%)      on eeeeee hsl(0°, 0%, 93%)     = EXAMPLE = 16.275:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg 111111 hsl(0°, 0%, 6%)      on 000000 hsl(0°, 0%, 0%)      = EXAMPLE =  1.112:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg 111111 hsl(0°, 0%, 6%)      on 111111 hsl(0°, 0%, 6%)      = EXAMPLE =  1.000:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
```

This is particularly much easier to use with a `| grep -v FAIL` at the end, to
show only combinations of foreground/background which _don't fail_ WCAG, i.e.:

```bash
$ for fg in $PALETTE; do for bg in $PALETTE; do ./contrast "$fg" "$bg" -l ; done ; done | grep -v FAIL
fg eeeeee hsl(0°, 0%, 93%)     on 000000 hsl(0°, 0%, 0%)      = EXAMPLE = 18.100:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg eeeeee hsl(0°, 0%, 93%)     on 111111 hsl(0°, 0%, 6%)      = EXAMPLE = 16.275:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg 000000 hsl(0°, 0%, 0%)      on eeeeee hsl(0°, 0%, 93%)     = EXAMPLE = 18.100:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
fg 111111 hsl(0°, 0%, 6%)      on eeeeee hsl(0°, 0%, 93%)     = EXAMPLE = 16.275:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text
```

Alternatively, you can use one (or more) of `-aa`, `-aaa`, `-AA`, `-AAA` to
specify your "target" WCAG level, and the program will try to find the smallest
change in lightness (or hue, see options) to make the contrast pass that level,
using a number of algorithms (by default all, can opt in to using only `--hsl`, 
`--oklab`, `--cielab`), and output the new foreground/background colors, the
contrast ratio, and the "EXAMPLE" text colored with the new colors, i.e.:

```bash
$ ./contrast a0a0a0 ca0000 -aaa -l
fg a0a0a0 hsl(0°, 0%, 62%)     on ca0000 hsl(0°, 100%, 39%)   = EXAMPLE =  2.287:1 contrast ✘FAIL AA  large ✘FAIL AAA large ✘FAIL AA  text ✘FAIL AAA text
fg c7c7c7 hsl(0°, 0%, 78%)     on 760000 hsl(0°, 100%, 23%)   = EXAMPLE =  7.017:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text BEST: algo=hsl    distance=0.3093
fg ffffff hsl(0°, 0%, 100%)    on 150000 hsl(0°, 100%, 4%)    = EXAMPLE = 20.351:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text       algo=oklab  distance=0.7309
fg d8d8d8 hsl(0°, 0%, 84%)     on 890000 hsl(0°, 100%, 26%)   = EXAMPLE =  7.140:1 contrast ✔PASS AA  large ✔PASS AAA large ✔PASS AA  text ✔PASS AAA text       algo=cielab distance=0.3183
```

As further options, when using one of `-aa`, `-aaa`, `-AA` or `-AAA` you can
specify `--keep-bg` or `--keep-fg` to keep that color fixed, and only change
the other one; `--hue-shift` to _allow_ changes in hue (by default only
lightness is changed); and `--ratio=F:B` (not allowed if `--keep-bg` or
`--keep-fg` is used) to specify a custom ligthness ratio to try when shifting
both foreground and background (by default 1:1, i.e. the same change in
lightness is applied to both foreground and background), i.e. `--ratio=2:1` to
try to change the foreground lightness twice as much as the background
lightness, or `--ratio=1:2` to try to change the background lightness twice as
much as the foreground lightness.

## LICENSE

Copyright 2023-2026 Marco Fontani <MFONTANI@cpan.org>

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice,
   this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors
   may be used to endorse or promote products derived from this software
   without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
