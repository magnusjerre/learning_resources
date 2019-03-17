# Source and .
`source` copies the contents of another file into the file calling `source`. This means that the copied contents is run in the same process. Calling `./<otherfile.sh>` in the caller file results in the "ohterfile" being called in a separate process / shell.
```bash
# fil 1: fil1.sh
NAME="Magnus"
echo "$NAME"

# fil 2: fil2.sh
. "fil1.sh" [arguments]
source "fil1.sh" [arguments]
# source and . are synonyms
echo "name again: $NAME"    # $NAME made available through fil1.sh
# Magnus
# name again: Magnus

# fil 3: fil3.sh
./fil1.sh
echo "name again: $NAME"    # $NAME is not made available through fil1.sh
# Magnus
# name again:
```
[Eksempel](https://superuser.com/questions/46139/what-does-source-do)

# base64 encoding / decoding
The argument is a file-name.
## Encoding
```bash
user$ base64 [filename] #returns the base64 encoded string
user$ echo "Magnus" | base64 #returns the base64 encoded string of the echo
TWFnbnVzCg==
```
## Decoding
```bash
user$ base64 -d [encoded file] #returns the decoded string of a file
user$ echo "TWFnbnVzCg==" | base64 -D #returns the base64 encoded string of the echo
TWFnbnVzCg==
```

# less, head, tail
These can be piped, though it's a little pointless.
## less
To search forwards for a pattern, first write a `/` followed by the pattern. To search backwards use `?` followed by the pattern. Press `n` to jump to next occurence, and `N` to jump to the previous occurence. To move up or down, use either `w`or __up arrow__ and `s` or __down arrow__. Press `G`to jump to the end. Press `g`followed by a line-number to jump to a given line. Press `F` to "follow" the file as it's updated. To show line numbers, include `-N` when using less. 

## head
Displays the first n-lines (defaults to 10). `head -n 20`displays the first 20 lines. Can take in more than one argument if wanted.

## tail
Displays the last n-lines. To continually follow the end of a file pass in `-f`. `-r`reverses the entire file.

## >> and >
`>>` appends to an existing file or creates a new one. `>` creates a new file or overwrites an existing one. `>` redirects output from one program to something other than stdout (which is the terminal by default).
```bash
$ ls > allmyfiles.txt # creates allmyfiles.txt filling it with the result of the ls command
$ echo "End of directory listing" >> allmyfiles.txt # appends "End of directory listing" to allmyfiles
$ ls > allmyfiles.txt # overwrites the content of allmyfiles.txt 
```

## | (pipe)
The pipe character `|` redirects the output from the left hand side into the input stream on the right hand side. 

## Running successive commands using only one line ; && ||
To run multiple commands, separate each command set with `;`, `&&` or `||`. Using `;` runs all commands whether any of them fail or not. `&&` shortcircuits so that the successive commands are only run if the previous command ran successfully. `||` behaves the same as in normal code.

## Useful navigational shortcuts
```
Ctrl-a: Move to the beginning of the line.
Ctrl-e: Move to the end of the line.
Ctrl-k: Delete the text from the cursor to the end of the line.
Ctrl-w: Delete the previous word.
Alt-d: Delete the next word.
Alt-f: Move forward a word.
Alt-b: Move backward a word.
Ctrl-n: Scroll down in history.
Ctrl-p: Scroll up in history
```

## Running a previous command with the same arguments
A shortcut to rerunning a previous command that had a long list of arguments is the prefix the command with `!`. Like so:
```bash
$ ls -ll
$ !ls # will do the same as the above "ls -ll"
```

Typing `history`will produce a list of the old commands. Rerun a command using `!n`, where n is the number of the command to run.

## Diffing files / streams
The `diff` command lets us show the difference between two files or directories. `-` means the "file" should be read from the standard input (stdin). Understanding the output:
* d: deletion
* a: addition
* c: changing

The number on the left is the line in file1, the number on the right is the line in file2. `3d2` means that the 3rd line in file1 was deleted and has the line number 2 in file2.

## Finding / Searching for a file
`find` lets ut walk a file hierarchy, it will recursively descend a directory tree. When looking at the manual, remember that the `expression` is comoposed of "primaries" and "operands". 
```bash
$ find ~/Documents -name "*txt" # Find all txt-files in the Documents folder
$ find -f dir1 dir2 -name "*txt" -and -name "*o*"   # Find all txt-files in dir1 and dir2 that end in txt and contains at least one "o"
$ find . -regex ".*1/y.*" # Normal regex on the entire path 
```

* -name: Search for filename, not path
* -iname: Same as name, but casesensitive
* -iregex: Regex on the whole path, caseinsensitive

# Replacing stuff in files or stdin using sed
`sed` lets us replace stuff using regex or basic regex. If no file is provided as an argument, `sed` will use stidn. By default the output is written to stdout, in order to write to a file we can send it using `>>`or `>`.
__Useful commands__
* d: Delete line. `sed '[adress] d' [filename]`
* s: Substitue apttern, `sed 's/[expattern]/[new pattern]/`
* [address]: The address pattern is either a single number, or range where start and end is seaparated by a comma. Postfix with ` !` to do something outside the given address range. The address can also be a regex pattern. `/^start/,/^end/ s/world/people/g'` will change world to people for every line after finding a line starting with "start", and stopping after finding a line starting with "end".

```bash
echo "http://www.jerre.no/kaffeapi/v2/allkaffe" | sed s/v2/v3/  # http://www.jerre.no/kaffeapi/v3/allkaffe
echo "mann som er sulten\ntrenger mat" | sed '1 d' #trenger mat
```
In order to access a regex-group in `sed` we use `\[group number]`.
```bash
$ echo "Magnus er sulten" | sed -E 's/(Mag[a-z]+)/\1 Jerre/' # Magnus Jerre er sulten
```
Note that `\d` and `\w` don't work in `sed` since they are regex-macros.
[sed-guide](https://adayinthelifeof.nl/2010/12/06/sed-simple-pattern-address-usage/)

## Word, line, character and byte count: wc
`wc` defaults to stdin if no file is given as an argument.

## grep
`grep -rnw . -e "[pattern]"` recursively searches for the files and lines that match the basic regex pattern. `grep -o [pattern]` prints only the matching part, not the entire line.