# MODULE I - LINUX SHELLS
## Lecture 1
Shell interface to get the OS to do things
  1. graphics
  2. command line (more versatile)

*We use bash in this course (`echo 0$` will tell you)*

#### Linux file system
  * `cat file` displays the content in a file
    - e.g. cat /user[the root]/arl[a directory]/words[the file]
  * `ctr + c` will kill the program
  * `ls` lists files in the directory
    - this only shows non-hidden files (names starts with a dot), `ls -a` will show all files

#### More on `cat`
  * just typing `cat` will repeat back everything you type
  * is this useful? maybe, if we can capture the output in a file!
    - `cat > output`
      - to stop: `ctr + d` at the beginning of a line (sents an EOF signal to `cat`)

*output redirection*: `command args > file`* (e.g. `cat -n > output.txt`) - output redirection

*input redirection*: `command args < file` (e.g. `cat < inputfile.txt` takes input from inputfile.txt instead of keyboard)

#### Misc
* `cat filename` passes the name filename as an arg to cat
  - cat opens the files and displays it
* `cat < filename` *the shell opens filename* and passes its contents to cat in place of keyboard
* `cat < in.txt > out.txt` copies in.txt to out.txt
* `wc filename` is a word count for filename
* `wc < filename` *the shell opens filename*
* `wc *.txt` - globbing pattern
  - \* = match any sequence of chars
  - shell finds all matching files in the current directory and substitutes it on the cmd line
* Every process is attracted to 3 streams:
  - stdin, stdout, stderr
  - input/output redirection connects stdin/stdout to a file
  - stderr is a *separate output stream* for error messages
    - so that output and errors are sent to different places, doesn't clutter output
  - Also: stdout may be **buffered** (the system may wait to accumulate output before actually displaying it)
    - stderr is never buffered

*can I use one program's stdout as another's stdin?*
  * Ex: how many words are in the first 5 lines of sample.txt
    - `head -n file` gives the first n lines of file
    - `wc` counts words, lines, chars, `wc -w` is just words
    - `head -5 sample.txt | wc -w` means pipe

## Lecture 2

Suppose `words1.txt`, `words2.txt`, etc. contain lists of words one per line. Print a duplicate free list of words that occur in any of these files.
  * `uniq` removes adjacent duplicate entries
  * `sort` sorts lines

Can we use the output of one prog as an argument of another? Ya.

`echo "today is $(date) and I am $(whoami)"` - the shell executes date and whoami and substitutes the result into the command line

#### On echo
  * The quotes group the arguments as one. If you were to execute `echo today is ...` you would have the same output, unless you have extra spaces then the spaces would be truncated without quotes.
  * Also, quotes suppress globbing. I.e. `echo *` will ls, while `echo "*"` will print a star
  * Single quotes are **even more** literal than double quotes - they will not do dollar sign expansion

#### Pattern-matching in text files
  * `grep/egrep` (extended global regular expression print)
  * `egrep pattern file` - print every line in file that contains pattern
  * e.g. print every line in index.html that contains cs246
    - `egrep 'cs246' index.html`
  * e.g. how many lines in index.html contain cs246 or CS246?
    - `egrep "cs246|CS246" index.html | wc -l` or `egrep "(cs|CS)246" ..`
  * Available patterns are regular expressions
  * `[..]` - any one char between [ and ]
  * `[^..]` - any one char **except**
  * Add optional space: `[cC][sS] ?246`
    - `?` means 0 or 1 of the preceding expression
  * `*` means 0 or more of preceding
  * `.` means any single char
  * `+` means 1 or more
  * `.*` literally anything
    - `egrep "cs.*246" index.html` means lines containing cs...anything...246
  * `^`, `$` match the beginning and end of a line

Examples:
  * Lines of even length: `egrep ^(..)*$`
  * Files in current dir whose names contain exactly one 'a': `ls | egrep ^[^a]*a[^a]*$`
  * All words in the global dictionary that starts with e and have 5 chars
    - `egrep "^e....$"`

#### Permissions
* `ls -l` - long form listing
  - `-rw-r----- 1 a42yu a42yu 25 sept 9 13:2 abc.txt`
  - [type]-[permission bits]  [num links] [file's owner] [file's group] [size] [last modified date and time] [filename]
  - groups: a user can belong to one or more groups, a file can be associated with one group
  - type: `-` means ordinary file, `d` means directory
  - permissions:
    - user bits: apply to the file's owner
    - group bits: members of the file's group (other than the owner)
    - other bits: everyone else
    - r: read bit
    - w: write bit
    - x: execute bit

| Bit | Meaning for ordinary files | Meaning for directories |
| --- | --- | -- |
| `r` | file's content can be read | directory's contents can be read e.g. ls works, globbing works|
| `w` | file's content can be modified | directory's contents can be modified i.e. you can add and remove files |
| `x` | file can be executed as a program | the directory can be navigated, i.e. you can cd into the dir|

If the dir's `x` bit is not set, there is no access **at all** to the dir, nor to any file or subdir within it

* Changing permissions: `chmod mode file`
  - modes:
    - `u` owner, `g` group, `o` other, `a` all
    - `+` add perm, `-` remove perm, `=` set perm exactly
    - `r` read, `w` write, `x` excecute
  - e.g. give others read perm: `chmod o+r file`
  - e.g. make everyone's permission `rx`: `chmod a=rx`
  - e.g. give the owner full control: `chmod u+rwx` or `chmod u=rwx`

#### Shell scripts
Files containing sequences of commands, executed as programs
  * E.g. print current date, user, directory
```bash
#!/bin/bash    <-shbang line, indicates that  it should run as bash script
date
whoami
pwd
```
Before you run it, set permissions: `chmod u+x myscript.bash`
To run it, you tell it to look at current dir: `./myscript.bash`

## Lecture 3

#### Variables
x=1 (**NO SPACES**)

  - you must use `$` when fetching the value of the variable
  - you *dont* use it when setting the variable
  - good practice: `${x}` - use brace brackets when fetching value

```bash
dir=~/cs246
echo ${dir}   # returns /u/a42yu/cs246
ls ${dir}     # gives you the content of dir
```
  - some global vars available
    - **important**: `${PATH}` is a list of dirs
      - when you type a command, the shell searches these dirs, in order, for a program of that name
    - `echo "${PATH}"` : (the path)
    - `echo '${PATH}'` : `${PATH}`
    - dollar expansion happens on double quotes, not single

#### Special variables for scripts
  - `$1`, `$2`, etc. - command line arguments
  - e.g. check whether a word is in the dictionary
    - `./isItAWord hello`

```bash
#!/bin/bash
egrep "^$1$" /usr/share/dict/words
```

  - e.g. a good password should not be in the dictionary, answer whether a word is a good password

```bash
#!/bin/bash
egrep "^$1$" /usr/share/dict/words > /dev/null    # /dev/null suppresses output
```
- every program returns a status code when finished
- egrep will return 0 if the pattern is found, or 1 if not
- in unix/linux: 0 means success, non-zero means fail
- `$?` is the status of the most recently executed command

example cont...
```bash
#!/bin/bash
egrep "^$1$" /usr/share/dict/words > /dev/null    # /dev/null suppresses output
if [ $? -eq 0 ]; then
  echo bad password
else # NOTE: elif is else if
  echo maybe a good password
fi

# ALTERNATIVELY:
if egrep "^$1$" /usr/share/dict/words; then .... fi
```

- e.g. verify the number of args; print error msg if wrong

```bash
#!/bin/bash
usage () {
  echo "usage: $0 password" >&2
}
if [ $# -ne 1 ]; then # $# is the number of args
  usage
  exit
fi
egrep blah blah blah
```

#### Loops
- e.g. print #s from 1 to $1

```bash
#/bin/bash
x=1
while [ $rx -le $1 ]; do
  echo $x
  x=$((x+1))
done
```

- e.g. rename all .cpp files to .cc

```bash
#!/bin/bash
for name in *.cpp; do # glob finds all matching files
  mv ${name} ${name%cpp}cc # value of name without the cpp
done
```

- e.g. how many times does the word $1 occur in file $2?

```bash
#!/bin/bash
x=0
for word in $(cat "$2"); do # round brackets prevents it from using cat $2 literally
  if [ $word = "$1" ]; then
    x=$((x+1))
  fi
done
echo $x
```

- e.g. payday is the last friday of the month. when is this month's payday?
  1. compute payday
  2. generate user friendly report on answer

```bash
#!/bin/bash
answer () { # NOTE: inside a function, ${1} etc are the args of the function
  if [ $1 -eq 31 ]; then
    echo "This month: the 31st"
  else
    echo "This month: the ${1}th"
  fi
}
answer $(cal | awk '{print $6}' | egrep "[0-9]" | tail -1)
```

- e.g. generalize to any month (e.g. cal October 2017)
  - `./payday October 2017` should give the right payday
  - `$(cal $1 $2 | awk '{print $6}' | egrep "[0-9]" | tail -1)`
    - if $1 and $2 are not provided, will just cal for the current month

```bash
#!/bin/bash
answer () { # NOTE: inside a function, ${1} etc are the args of the function
  if [ $2 ]; then # month is not blank
    prefix="This month"
  else
    prefix=$2
  fi

  if [ $1 -eq 31 ]; then
    echo "$prefix: the 31st"
  else
    echo "$prefix: the ${1}th"
  fi
}
answer $(cal $1 $2 | awk '{print $6}' | egrep "[0-9]" | tail -1) $1
```

# SE Topic: Testing ❎
- essential, ongoing part of program development
  - begins before you start coding 😆
  - test suites - expected behaviour
- debugging - can't debug without first testing
- can't guarantee correctness, can only prove wrongness
- ideally, developer and tester should be different people
- human testing: humans look over code, find flaws
  - code inspections, walkthroughs
- machine testing: run program on selected input, check output against spec
  - can't check everything, must check carefully
- black/white/grey box testing
  - no/full/some knowledge of program input
  - various classes of input, e.g. number ranges, positive vs negative
  - boundaries of valid data ranges (edge cases)
  - multiple simultaneous boundaries
  - intuition / experience
  - do black box first, then do white box later
- whitebox
  - execute all logical paths through te program
  - make sure every function runs
- performance testing - is the program fast enough
- regression testing - make sure the new changes to the program don't break the old test classes
  - use test suites
