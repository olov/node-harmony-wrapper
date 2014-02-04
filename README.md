# node-harmony-wrapper

So you want to create a wrapper for your node program.

If your program does not require any features hidden behind the `--harmony` flag then it is as
simple as using a [shebang](http://en.wikipedia.org/wiki/Shebang_(Unix)).
[hello-es3.js](hello-es3.js) contains our  program and [hello-es3](hello-es3) has a `#!/usr/bin/env
node` shebang and requires the `.js` file. Simple as that! You can of course slap the shebang
directly on top of [hello-es3.js](hello-es3.js) if you like that better.

For a program that needs `--harmony`, such as [hello-es6.js](hello-es6.js), things get complicated.
In the best of worlds you would have been able to just change the shebang into `#!/usr/bin/env node
--harmony` and things would have worked. And it does, on Mac. But it does not on Linux. Because
POSIX never specified shebang arguments. Oh Candide! :(

So now what do you do? Short answer: Copy and rename [hello-es6](hello-es6).

Optional long answer: What we usually do, you add another level of indirection. A shell script
wrapper to the rescue. It should merely execute `/usr/bin/env node --harmony "$DIR/hello-es6.js"
"$@"` where `$DIR` is the path to the program directory (and in this case, also the wrapper
directory). Finding that path is a bit trickier than it may first seem, because of symlinks. The
program may have been invoked through any number of absolute or relative symlinks, and any path
(symlink or not) may be made up of one or more symlink'ed directories. If we could just resolve any
number of symlinks possibly hiding in `$0`, and then chop of the trailing non-directory part of
that, we would have our `$DIR` variable.

One would think that recursively resolving a symlink would be a common enough operation to be
POSIXed, but we have no such luck. On Linux `readlink -f` is exactly what we need but on Mac, we
have to do with plain non-recursive `readlink`. So a shell function `rdlkf()` ("readlinkf") must be
created to mimic that. Once defined, `DIR="$(dirname "$(rdlkf "$0")")"` should do the trick.

If you're curious about why we've turned the quote-knob up to eleven the answer is because
whitespace.

The final shell wrapper can be found in [hello-es6](hello-es6).

Shell scripting is terrible. How terrible you ask? [Take a look](hello-es6) for yourself.

PR's welcome but make sure you're actually testing various symlink-scenarios before you golf
`rdlkf`.

Disclaimer: "on Mac" and "on Linux" is not a precise description of the entire UNIX/POSIX landscape
and the wrapper is mostly not specific to node.
