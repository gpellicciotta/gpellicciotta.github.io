# Workarounds for (Un)Common Software Issues

## Python Setup on Windows, with Cygwin

### Issue
Mixing Python on Windows, with Cygwin installed is a pain:
- Native Windows Python doesn't recognize cygwin paths (e.g. /cygdrive/c)
- Cygwin python doesn't deal with a ;-separated PYTHONPATH environment variable
So how can one work with both?

### Ideal Solution
The ideal is:
- Shebang line that should work everwhere, also on linux/unix: `#!/bin/env python` was would be ideal
- `PYTHONPATH` var that must only be set once
- Ability to start python scripts from batch file or shell script on Windows, and from shell script on linux/unix

### Workaround
My current work-around involves following setup:

- (Make sure .py files are started by the Windows PYTHON automatically by file association - Windows doesn't know about shebang lines)
  - Make sure Windows PYTHON comes before Cygwin/bin dir in `PATH` env. variable
  - Ensures that Windows PYTHON will be found by cmd prompt and bat files
  - Shell scripts and bash shell will still find Cygwin PYTHON as they always prepend the `PATH` with the Cygwin/bin dirs
- Add following line to my .bashrc file, which ensures the PYTHONPATH must only be set once, and it must be set in true Windows style, 
  with Windows paths and separated by semicolons:
  ```
    export PYTHONPATH=`echo $PYTHONPATH | sed -e 's#[\\]#/#g' | sed -e 's@^\([A-Za-z]\):@/cygdrive/\1@g' | sed -e 's@;\([A-Za-z]\):@;/cygdrive/\1@g' | sed -e 's@;@:@g'`
  ```  