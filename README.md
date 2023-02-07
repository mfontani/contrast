# contrast

A program to check WCAG contrast rules for foreground/background colors.

I've had this in my home toolbox for years, but recently needed this on another
machine, so I'm making it available so I can find it again.

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

## LICENSE

Copyright 2023 Marco Fontani <MFONTANI@cpan.org>

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
