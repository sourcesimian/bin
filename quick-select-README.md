quick-select  <!-- omit in toc -->
============

***A utility that makes selecting an item from a list quick and easy***

- [Usage](#usage)
- [Example Use Cases](#example-use-cases)
  - [Basic List Selection](#basic-list-selection)

# Usage
```
usage: quick-select [-h] [-t TITLE] [-p PROMPT] [-v VALUE_FILE] list

positional arguments:
  list                  A CSV file with "Value[,Display[,Extra]]" fields from which the selection is to be made

optional arguments:
  -h, --help            show this help message and exit
  -t TITLE, --title TITLE
                        Text displayed at top of terminal
  -p PROMPT, --prompt PROMPT
                        Text that prefixes the match entry at bottom of terminal
  -v VALUE_FILE, --value-file VALUE_FILE
                        Write the selected value to this file

For a demo run: quick-select demo
```

# Example Use Cases
## Basic List Selection
We start with our list of items:
```bash
$ cat ./items.txt
Viola
Ukulele
Vocals
Bells
Steel Pan
...
```

Then we run `quick-select` like this:
```bash
$ quick-select ./items.txt
```

and shows us the following screen:
```
Make your selection and hit RETURN
┌───────────────────────────────────────────────────────────────┐
│*Viola*         Triangle        Cowbell         Organ          │
│Ukulele         Bongos          Crystal Glasses Turntables     │
│Vocals          Drums           Vibraphone      Piano          │
│Bells           Stylophone      Ocarina         Shakers        │
│Steel Pan       Piccolo         Bagpipes        Slide Whistle  │
│Acoustic Guitar Bass Guitar     Saxophone       Accordion      │
│Flute           Harp            Tuba            Spoons         │
│Trombone        Keyboard        Violin          Oboe           │
│Trumpet         Whistle         Xylophone       Recorder       │
│Steel Drums     Guitar          Voice           Bamboo Flute   │
└───────────────────────────────────────────────────────────────┘
 Match >
```

Now using the arrow keys you can move the cursor to your chosen item and hit RETURN.
Or you can type some fragment of the item you want to select and the list will reduce, for example here we typed `ph`:
```
Make your selection and hit RETURN
┌───────────────────────────────────────────────────────────────┐
│*Stylophone*                                                   │
│Vibraphone                                                     │
│Saxophone                                                      │
│Xylophone                                                      │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
└───────────────────────────────────────────────────────────────┘
 Match > ph
```

Now the list is much shorter. You can move the cursor to your chosen item and hit RETURN. Or you can type some other fragment, for example `i`:
```
Make your selection and hit RETURN
┌───────────────────────────────────────────────────────────────┐
│*Vibraphone*                                                   │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
│                                                               │
└───────────────────────────────────────────────────────────────┘
 Match > ph i
```

Now you have just one match and hit RETURN.
```
$ quick-select ./items.txt
Vibraphone
```

Now you can use the the following idiom to capture the output value:
```bash
if quick-select -v /tmp/quick-select-value ./items.txt; then
    VALUE=$(/tmp/quick-select-value)
    ....
else
    ...
fi
```

## Quick SSH
If you happen to have a large number of SSH hosts configured in your `~/.ssh/config` file, here is a quick way of getting the host you want. Save this to a script called `qssh`:
```bash
#!/bin/bash
set -e

EXEC=/tmp/quick-select
URL=https://raw.githubusercontent.com/sourcesimian/bin/main/quick-select
VALUE=/tmp/qssh-value

if [ ! -e $EXEC ]; then
    echo "# Fetcing quick-select to $EXEC"
    curl -ks -o $EXEC $URL
    chmod +x $EXEC
fi

if $EXEC -v $VALUE <(grep "^Host" ~/.ssh/config | grep -v -F '*' | cut -d' ' -f2- | tr ' ' '\n' | cat -s | uniq); then
    echo "# ssh $(cat $VALUE) ${@}"
    exec ssh $(cat $VALUE) "${@}"
fi
```
