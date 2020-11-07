# sshx - Run a Local Script over SSH with Interactivity
Sometimes you want to run a bit more than just one command over ssh, and you also want to maintain interactivity to the processes that you launch. Usually you could just do
```ssh -t <host> "foo; bar; fish; paste"```
however that can quickly become unwieldly, especialy if it is more than just a sequence of commands. You also can't use
```cat my_script | ssh <host> /bin/bash```
because then you loose your `stdin` and thus your interactivity.

However, there is a way to achieve our goal using the first form above. Consider that the maximum command line length is usually more than 2MB (see `getconf ARG_MAX`). Thus, we convert our entire script into a base64 encoded string with `cat my_script | base64` and then call
```ssh -t <host> /bin/bash "<(echo <base64 string> | base64 --decode)"```.
In this way the entire script is sent to the host in the command buffer, and so long as our total command length is less than 2MB, it will run.

Note: the usage of the `<(...)` ([Process Substitution](https://www.tldp.org/LDP/abs/html/process-sub.html)), and the quotes to prevent it from running locally.

So, now expressing that in full form would be:

```ssh -t <host> /bin/bash "<(echo "$(cat my_script | base64)" | base64 --decode)" <arg1> ...```

Though to avoid using the echo, which feels a bit klunky, lets rather use a `<<EOF` ([Here Document](https://www.tldp.org/LDP/abs/html/here-docs.html)) with `cat` and also use `$'...'` ([ANSI C Quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html)) for precise control, which gives us:

```ssh -t <host> /bin/bash $'<(cat<<_ | base64 --decode\n'$(cat my_script | base64 | tr -d "\n")$'\n_\n)' <arg1> ...```

> Edit: a shorter form not using `cat` and making use of a [Here String](https://www.gnu.org/software/bash/manual/html_node/Redirections.html) gives a:
> ```ssh -t <host> /bin/bash '<(base64 --decode <<<'$(cat my_script | base64 |tr -d "\n")')'```

Note: the `tr -d "\n"` is helpful as some base64 decode implementations will fail on the whitespace. I used a `_` as the EOF marker as it is not a character in the base64 set.

This idea is developing quite nicely, however it still falls short in some ways. What if say we want to run a Python script? We'd have to change to `ssh -t <host> /usr/bin/env python ...`. And if we wanted to provide arguments to our script that are actually input files, how would we get those across the wire?

Considering the above and more, after some further tinkering we can arrive at **sshx** (see full source below) which will marshall up an entire command line and run it over ssh as follows.

Say we have a Python script called "**foo**":
```
#!/usr/bin/env python
import sys
arg1 = sys.argv[1]
print "The contents of arg1:" 
sys.stdout.write(open(arg1).read())
```
And a file called "**hi**":
```
Hello World!
```
To run `$ foo hi` on a remote host we could generate the following BASH fragment:
```
export SCRATCH=$(TMPDIR=$HOME mktemp -d -t .scratch.XXXXXXXX)
trap "{ rm -rf \"$SCRATCH\"; }" EXIT SIGINT SIGTERM SIGKILL
chmod 700 $SCRATCH
# cmd: foo hi
# file: "foo" -> "$SCRATCH/foo"
cat<<_ |base64 --decode >"$SCRATCH/foo"; chmod 755 "$SCRATCH/foo"; touch -d @1557867114 "$SCRATCH/foo"
IyEvdXNyL2Jpbi9lbnYgcHl0aG9uCmltcG9ydCBzeXMKYXJnMSA9IHN5cy5hcmd2WzFdCnByaW50ICJUaGUgY29udGVudHMgb2YgYXJnMToiIApzeXMuc3Rkb3V0LndyaXRlKG9wZW4oYXJnMSkucmVhZCgpKQo=
_
# file: "hi" -> "$SCRATCH/hi"
cat<<_ |base64 --decode >"$SCRATCH/hi"; chmod 644 "$SCRATCH/hi"; touch -d @1557867136 "$SCRATCH/hi"
SGVsbG8gV29ybGQhCg==
_
$SCRATCH/foo $SCRATCH/hi 
```
Then we further base64 encode this marshalled command and use the above technique to run the command over ssh.

**sshx**:
[sshx](./sshx)
