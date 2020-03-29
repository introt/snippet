# revamp

```revamp``` is an advanced template based string formatting tool which 
has the ability to do complex variable substitutions:

```
$ revamp -f "echo '<arg1> <arg2>';" hello revamp
echo 'hello revamp';
```

Besides copy and pasting the output where you actually need it you can pipe the result to a command processor 
which allows direct and easy execution of alternating commands:   
```
$ revamp -f "echo '<arg1> <arg2>';" hello revamp | bash
hello revamp
```


## Assigning data to placeholders
To assign data to a placeholder you have several options:

### Using positional arguments
The most straight forward way of assigning data to any unset placeholder is by using positional arguments:

```
$ revamp -f "echo 'hello <arg1>';" revamp
echo 'hello revamp';
```
To assign multiple values to one specific placeholder values need to be enclosed in ```\(``` and ```\)```:

```
$ revamp -f "echo 'hello <arg1>';" "\('revamp' 'world'\)"
echo 'hello revamp';
echo 'hello world';
```

### Using environment variables

```revamp``` evaluates environment variables and assigns data to any unset 
placeholder. To avoid running into conflict with other environment variables ```revamp``` only evaluates 
lower-case variable names:
```
$ export arg1=revamp
$ revamp -f "echo 'hello <arg1>';"
echo 'hello revamp';
```
To assign multiple values to a placeholder following syntax needs to be used:
```
$ export arg1="\('revamp' 'world'\)"
$ revamp -f "echo 'hello <arg1>';"
echo 'hello revamp';
echo 'hello world';
```

### Using the --set argument

Another option to assign data is using the ```-s | --set``` argument: 
```
$ revamp -f "echo 'hello <arg1>';" -s arg1=revamp
echo 'hello revamp';
```
Similar to environment variables the ```-s | --set``` argument allows assigning multiple values to a placeholder:
```
$ revamp -f "echo 'hello <arg1>';" -s arg1="\('revamp' 'world'\)"
echo 'hello world';
echo 'hello revamp';
```
Another option is to use the ```-s | --set``` argument several times in a row:
```
$ revamp -f "echo 'hello <arg1>';" -s arg1=revamp -s arg1=world
echo 'hello world';
echo 'hello revamp';
``` 
In addition the ```-s | --set``` argument allows importing values from a file:
```
$ cat <<EOF > input.txt
revamp
world
EOF
$ revamp -f "echo 'hello <arg1>';" -s arg1:input.txt
echo 'hello world';
echo 'hello revamp';
```

### Importing csv-files

Data can also be imported from a csv-file using the ```-i | --import``` argument.
Note, that values must be separated by a tab character
 (which can be changed in ```.revamp/revamp_profile.py```):

```
$ cat <<EOF > input.csv
arg1    arg2
hello   revamp
        world
EOF
$ revamp -f "echo '<arg1> <arg2>';" -i input.csv 
echo 'hello world';
echo 'hello revamp';
```

### Presets

```revamp``` ships with a customizable set of preset placeholders which can be 
directly used in your format string 
(see ```.revamp/revamp_profile.py``` for more information). Preset placeholders may have constant  
but also dynamically generated values assigned to them:
```
$ revamp -f "echo '<date_time>';" 
echo '20200322034102';
```

## Using string formats

To use string formats you have several options:

### Using the --format-string argument

If you read the previous section you already know the ```-f | --format-string``` argument:

```
$ revamp -f "echo 'hello revamp';"
echo 'hello revamp';
```

### Environment variables
```revamp``` allows setting the string format using the ```FORMAT_STRING``` environment variable:
```
$ export FORMAT_STRING="echo 'hello revamp';"
$ revamp
echo 'hello revamp';
```

### Templates

If you want to persist your format string you can add them to ```revamp```'s template directory
which can be easily accessed using the ```-t | --template``` argument:
```
$ mkdir -p ~/.revamp/templates
$ echo -n "echo '<arg1> <arg2> - <date_time>'" > ~/.revamp/templates/example
$ revamp -t example -s arg1=hello -s arg2=revamp
hello revamp - 20200322034102
```

If you have bash-completion enabled you can press ```<TAB>``` twice to autocomplete 
template names. 

If you want to review a specific template you can use the ```-v | --view-template``` argument:
```
$ revamp -t example -v
echo '<arg1> <arg2> - <date_time>'
```

To list all available templates you can use the ```-l | --list-templates``` 
parameter:

```
$ revamp -l
net/enum/enum4linux-basic
net/scan/nmap-basic
net/scan/nmap-ping
net/scan/nmap-syn
net/scan/nmap-version
os/exec/bash-curl
os/exec/bash-wget
os/exec/cmd-webdav
os/exec/powershell-http
os/exec/powershell-webdav
os/exec/xargs-parallel
shell/listener/nc
shell/listener/ncat
shell/listener/netcat
shell/listener/socat
shell/reverse/linux/groovy
shell/reverse/linux/java
shell/reverse/linux/perl
shell/reverse/linux/php
shell/reverse/linux/python
shell/reverse/linux/ruby
shell/reverse/multi/powershell
shell/reverse/multi/python
shell/reverse/multi/socat
shell/reverse/windows/groovy
shell/reverse/windows/perl
shell/reverse/windows/ruby
shell/upgrade/pty
web/fuzz/gobuster-basic
web/fuzz/nikto-basic
web/fuzz/wfuzz-basic
web/fuzz/wfuzz-ext
```

## Command Execution

```revamp``` can be used to easily execute alternating commands in sequence:
```
$ revamp -f "echo 'hello <arg1>';" -s arg1=revamp -s arg1=world | bash
hello world
hello revamp
```

Using ```xargs``` the resulting commands can also be executed in parallel:
```
revamp -f "echo 'hello <arg1>';" -s arg1=revamp -s arg1=world | xargs --max-procs=4 -I CMD bash -c CMD
hello world
hello revamp
```

## Configuration
To enable bash-completion add following line to your .bashrc:
```bash
eval "$(register-python-argcomplete revamp)"
```
Make sure to add the ```revamp``` directory to your ```PATH``` or link it accordingly:
```
ln -s /path/to/revamp.py /usr/bin/revamp
```

