# sshx - Run a Local Script over SSH

Usage:
```
SSHX: Run a Local Script over SSH

Usage:  sshx [options] -- [command]

Options:
  *                SSH options
  --workdir=<dir>  Change to directory before running command
  --debug          Run the wrapper script with debug output
  --dump[=<file>]  Write the wrapper script to the file. Default /dev/stdout

Command:
  The command and arguments you wish to run on the remote host. This may
  include a local script and/or local argument files.

A SSH wrapper script that serialises a local command/script and runs it on
a remote host. All arguments which are local files are serialised and made
available on the remote host. The total size of the command/script and file
arguments must be less than ARG_MAX since the upload is done in the SSH
command line buffer.
```

## Example Usage

Run a local script on a remote host while preserving full I/O and exit code:
```
sshx host -- ./my_script
```

Which means that you can do:
```
sshx host -- ./my_script  < ./my_input  > ./my_output  2> ./my_error; echo $?
```

Additionally you can provide local files as arguments too:
```
sshx host -- ./my_script  -c ./my_config
```

And you can also choose the working dir where the command is run:
```
sshx host --workdir=~/projects/foo -- ./my_script
```

## Limitations
The implementation of `sshx` is based on base64 encoding the uploaded content
onto the SSH command line, typically the maximum size is usually larger than 128kB,
and often up to 2MB. Running `getconf ARG_MAX` will give us a rough idea of how
much space we have available. If you exceed the limit you should get an
`Argument list too long` error.

Out of interest you can use the `--dump` option to output the wrapper script
that will get run on the remote host. Or you can use the `--debug` option to
see the BASH debug output of the wrapper script as it runs.

## Source
Here is [sshx](./sshx)
