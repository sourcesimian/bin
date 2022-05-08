# sshx - Run a Local Script over SSH

Usage:
```
SSHX: Run a Local Script over SSH                     v1.0   by Source Simian

A SSH utility script that serialises a local command including the contents of
any script and argument files and runs it on a remote host. The total size of
the command and file arguments must be less than ARG_MAX since the upload is
done in the SSH command line buffer.

Usage:  sshx [options] -- [command]

Options:
  *                SSH options
  --workdir=<dir>  Change to directory before running command
  --env=<name>     Export local environment variable to the remote host
  --rcfile=<file>  Source this file before running the command
  --gzip           Compress the launcher script content with Gzip
  --debug          Run the launcher script with debug output
  --dump[=<file>]  Write the launcher script to the file. Default /dev/stdout

Command:
  The command and arguments you wish to run on the remote host. This may
  include a local script and/or local argument files.

Source:
  https://github.com/sourcesimian/bin
```

## Source
Here is [sshx](./sshx)

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
`Argument list too long` error. If `gzip` is installed locally and `gunzip` is
installed on the remote host you can try the  `--gzip` option.

Out of interest you can use the `--dump` option to output the launcher script
that will get run on the remote host. Or you can use the `--debug` option to
see the BASH debug output of the launcher script as it runs.

## Installation
Feel free to install however you like, it is just the one file. Personally I
checkout this repo and symlink to my `~/bin/`. For your convienience here is a
quick way to install it:
```
curl -o ~/bin/sshx https://raw.githubusercontent.com/sourcesimian/bin/main/sshx
chmod +x ~/bin/sshx
```
