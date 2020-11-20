pexec-to-file  <!-- omit in toc -->
=============

***A helper to run scripts in parallel***

- [Usage](#usage)
- [Example Use Cases](#example-use-cases)
  - [Running Locally](#running-locally)
  - [Running Remotely over SSH](#running-remotely-over-ssh)

# Usage
```
pexec-to-file <thread count> <pattern> <out file> <args ...>
```

# Example Use Cases
## Running Locally
We start with our list of items:
```
$ cat ./items.txt
marmoset
lovebird
duckbill platypus
aardvark
cow
hedgehog
```

Now, `pexec-to-file` accepts the items on stdin so:
```
$ cat ./items.txt | pexec-to-file 3 {} out-{}.txt echo "Hello: {}"
# echo Hello:\ marmoset &>out-marmoset.txt &
# echo Hello:\ lovebird &>out-lovebird.txt &
# echo Hello:\ duckbill\ platypus &>out-duckbill platypus.txt &
# echo Hello:\ aardvark &>out-aardvark.txt &
# echo Hello:\ cow &>out-cow.txt &
# echo Hello:\ hedgehog &>out-hedgehog.txt &
```

And we get the output files:
```
$ ls -1 out-*.txt
out-aardvark.txt
out-cow.txt
'out-duckbill platypus.txt'
out-hedgehog.txt
out-lovebird.txt
out-marmoset.txt
```

and the contents of the output files:
```
$ grep '.*' out-*.txt
out-aardvark.txt:Hello: aardvark
out-cow.txt:Hello: cow
out-duckbill platypus.txt:Hello: duckbill platypus
out-hedgehog.txt:Hello: hedgehog
out-lovebird.txt:Hello: lovebird
out-marmoset.txt:Hello: marmoset
```

## Running Remotely over SSH
Let's say we have a have a script that we wish to run on several hosts, for this example we'l keep it trivial:
```
$ cat ./hello.sh
#!/bin/bash

echo "Hello from: $(cat /etc/hostname)"
```

We have our hostnames in a list:
```
$ cat ./hosts.lst
cluster-node-1
cluster-node-5
cluster-node-7
cluster-node-11
cluster-node-12
```

Now using the same techinque we used in Running Locally above, and additionally using  [sshx](./sshx-README.md):
```
$ cat hosts.lst | pexec-to-file 3 {} out-{}.txt sshx {} ./hello.sh
# sshx cluster-node-5 ./hello.sh &>out-cluster-node-5.txt &
# sshx cluster-node-1 ./hello.sh &>out-cluster-node-1.txt &
# sshx cluster-node-7 ./hello.sh &>out-cluster-node-7.txt &
# sshx cluster-node-11 ./hello.sh &>out-cluster-node-11.txt &
# sshx cluster-node-12 ./hello.sh &>out-cluster-node-12.txt &
```

And we get the output files:
```
$ ls -1 out-*.txt
out-cluster-node-11.txt
out-cluster-node-12.txt
out-cluster-node-1.txt
out-cluster-node-5.txt
out-cluster-node-7.txt
```

and the contents of the output files:
```
$ grep '.*' out-*.txt
out-cluster-node-11.txt: Hello from: 10.0.0.11
out-cluster-node-12.txt: Hello from: 10.0.0.12
out-cluster-node-1.txt: Hello from: 10.0.0.1
out-cluster-node-5.txt: Hello from: 10.0.0.5
out-cluster-node-7.txt: Hello from: 10.0.0.7
```
