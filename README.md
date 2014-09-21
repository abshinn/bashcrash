BASHCRASH
===

A crash course in bash for the data scientist.


### Unix Philosophy

"This is the Unix philosophy: Write programs that do one thing and do it well. Write programs to work together. Write programs to handle text streams, because that is a universal interface." -- [Doug McIlroy](http://www.faqs.org/docs/artu/ch01s06.html)


### Basics

List directory contents with `ls`, create a directory with `mkdir`, copy files with `cp`, move and rename files with `mv`, remove files and directories with `rm` and `rmdir`.

Most importantly, access the Bash manual `man` to get help with built-in commands.


#### Keyboard shortcuts

Command-line keyboard shortcuts are immensely useful. For example, `ctrl+a` moves the cursor to the beginning of the line and `ctrl+e` moves the cursor to the end. Additionally, `ctrl+k` clears the line after the cursor, and `ctrl+l` clears the whole terminal window -- equivalent to `clear`.

See a list of shortcuts here: [Wikipedia article on Bash, shortcut section](http://en.wikipedia.org/wiki/Bash_(Unix_shell)#Keyboard_shortcuts)


#### Optional matching, brace expansion, and !$

Not only can you use the `*` wildcard, but bash also supports optional matching. Show all tsv and csv files in the current directory:

```bash
ls -l *.[tc]sv
```

Expand items with brackets `{}`. Only list filenames with certain extensions:

```bash
ls -l *.{csv,tsv,sql,json}
```

Another useful feature is the ability to repeat the last argument with `!$`. Show all csv files, then move them to a directory, `data/`:

```bash
ls -l *.csv
mv !$ data/
```

#### Find

The `find` command has several different applications. 

To search all sub-directories within a directory called `zipfian` for python files:

```bash
find ~/zipfian -name ".py"
```

The find command can also execute a given command on all of the files returned by the search using the `-exec` flag. To search for all python files within the `zipfian` directory that contain the string `"RandomForestClassifier"`:

```bash
find ~/zipfian -name "*.py" -exec grep -l "RandomForestClassifier" {} \;
```

Many more examples on the `find` command here: [Blog post by Alvin Alexander](http://alvinalexander.com/unix/edu/examples/find.shtml)


### Environment

#### Where does Bash search for commands?

```bash
echo $PATH
```

`$PATH` is a system variable string which contains paths separated by colons `:`. Bash searches the paths in `$PATH`, precedence goes to the command left-most path if you have more than one command in the search path. 

It is very common for `$PATH` to be defined or appended within your shell initialization script. By default, that script is `~/.bash_profile`. If you are using [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh), then your initialization script is `~/.zshrc`.

#### Where are commands located in the system?

The very basic system commands are located in `/bin` and `/usr/bin/` directories.

Have a look at those commands using `ls`:

```bash
ls -l /usr/bin
```

Or, search those commands by piping the above output to `grep`. In the below example, I search for the word `"grep"` to search for all of the regular expression commands related to `grep`.

```bash
ls -l /usr/bin | grep "grep"
```

Note that the quotation marks are not necessary. Bash knows that the pattern argument passed into `grep` is a string.

To show the origin of a specific command, link, or alias, you can use either `which` or `type`. Also, most properly implemented commands and interpreters have a `--version` flag that will display the version of the command.

```bash
which python
python --version
```

#### Scripting

`#!` The hashbang! Also known as the shebang.

_example python script: environ.py_

```python
#!/usr/bin/env python

import os

for item in os.environ.items():
    print item
```

Supplying `#!/usr/bin/env python` in first line of a script will tell Bash to pass the script as an argument to your current environment's version of `python`. In other words, it is equivalent to running `python environ.py` in your current shell.

But, there's one more step. That is to add the executable mode to your file using `chmod +x`.

```bash
chmod +x environ.py
```

Now, the file can be executed by typing out its relative or absolute path. In the current working directory, that will be `./`+`filename`.

```bash
./environ.py
```

For Bash scripts, it is common to just provide the path to `bash` in the shebang.

_example bash script: environ.sh_

```bash
#!/bin/bash

cat printenv > environment.txt

echo "environment variables stored in environment.txt"
```

Learn more about shebang here: [Shebang wikipedia article](http://en.wikipedia.org/wiki/Shebang_(Unix))

Learn more about writing scripts in the Bash language here: [Robert Kline's Bash Tutorial](http://www.cs.wcupa.edu/rkline/linux/bash-basics.html#execute)


### Process Management

#### ; and &

To run commands in a batch, separate them by semicolons:

```bash
command1 ; command2
```

Or, to run both commands concurrently:

```bash
command1 & command2
```

Note: `command1` starts running in the background, and `command2` runs in the foreground.

#### ctrl, bg, fg, and kill

To quit a process running in the foreground with `ctrl+c` or in some cases, `ctrl+d`.

Suspend a process running in the foreground with `ctrl+z`. You will see something like this:

```
^Z
[2]  + 71826 suspended  ./energy.py
```

Resume a suspended process in the foreground with `fg`. Or you can run it in the background with `bg`, which is equivalent to running `./energy.py &`.

Use `jobs -l` to list ongoing and suspended jobs with their pid (process id). The output looks something like this:

```
[1]  - 71979 running    ./pima.py
[2]  + 71826 suspended  ./energy.py
```

Finally, you can quit that suspended process with `kill`:

```bash
kill 71826
```

#### ps and top

To display all current shell window processes use `ps`, and to see all current system processes use `ps ax`. Or, to see what process is eating up all of your memory have a look at the `top` command.

#### purge

After many hours of uptime and usage your computer can build up inactive memory. Darwin OS's `purge` simulates restart conditions by flushing disk cache.


### Working with CSV

#### Manipulation with sed and awk

When CSVs are improperly formatted, sed and awk can come to the rescue. For help on crafting regular expressions, see [Regexr](http://www.regexr.com/).

A somewhat useful blog post on csv manipulation: [Sultan of Awk](http://abshinn.github.io/bash/2014/05/17/sultan-of-awk/)

#### CSVkit

CSVkit is a super-useful collection of commands that operate on CSVs. It is implemented in python, and can be easily installed with:

```bash
pip install csvkit
```

#### Displaying CSVs with csvlook


To display a csv in an easy-to-read format, use `csvlook`:

```bash
cat example.csv | csvlook
```

If it is a csv with many columns and rows, this may not be a pretty output. In that case, you can then pipe the output to `less -S` for easier viewing. The `-S` flag tells `less` not to fold lines.

```bash
cat example.csv | csvlook | less -S
```

Furthermore, you can choose columns with `csvcut` and sort on specified columns with `csvsort`.

#### Summary statistics on the command line

Much like with Pandas' describe method, you can run summary statistics on a CSV with the use of `csvstat`.

For more information, see: [CSVkit documentation](https://csvkit.readthedocs.org/en/0.9.0/)


### Working with JSON

Another super-useful tool for displaying and manipulating data on the command line is JQ. You can download JQ with homebrew:

```bash
brew install jq
```

Or, if you don't have homebrew, you can find it here: [JQ home](http://stedolan.github.io/jq/)

JQ can pretty-print, extract, and manipulate JSON data on the command line.

```bash
cat example.json | jq ".[0]"
```


### Recommended shell, zsh
- [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
- [my zshrc profile](https://github.com/abshinn/oh-my-zsh/blob/master/abshinn.zshrc)


### Other Useful Links
- [Regexr](http://www.regexr.com/)
- [7 command-line tools for data science](http://jeroenjanssens.com/2013/09/19/seven-command-line-tools-for-data-science.html)
- [Data Science at the Command Line](http://datascienceatthecommandline.com/)
