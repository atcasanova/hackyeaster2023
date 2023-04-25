# Ghost In A Shell 4

## Description
```
  _, _,_  _,  _, ___   _ _, _    _,    _, _,_ __, _,  _,    , ,   ,  
 / _ |_| / \ (_   |    | |\ |   /_\   (_  |_| |_  |   |     | \   /  
 \ / | | \ / , )  |    | | \|   | |   , ) | | |   | , | ,   |  \ /  
  ~  ~ ~  ~   ~   ~    ~ ~  ~   ~ ~    ~  ~ ~ ~~~ ~~~ ~~~   ~   ~   
______________________________________________________________________  
 ,--.     ,--.     ,--.     ,--.  
| oo |   | oo |   | oo |   | oo |  
| ~~ |   | ~~ |   | ~~ |   | ~~ |  o  o  o  o  o  o  o  o  o  o  o  o  
|/\/\|   |/\/\|   |/\/\|   |/\/\|  
______________________________________________________________________  
```
  
Connect to the server, snoop around, and find the flag!

ssh ch.hackyeaster.com -p 2306 -l blinky
password is: blinkblink
Note: The service is restarted every hour at x:00.

## Solution

This one might seem simple, but my solution is a bit of a hidden gem that not many people know about. So let's dive into this!

After snooping around, you quickly realize that almost no command seems to work. If you're in the game long enough, you'll know that there are a lot of aliases blocking your way.

<img width="454" alt="image" src="https://user-images.githubusercontent.com/2973929/234145852-800a2628-86f7-47fe-ace5-19e962430f02.png">

Now, you could just "unalias" each of them and grab the flag, but wait! There's a cool trick:

<img width="149" alt="image" src="https://user-images.githubusercontent.com/2973929/234146129-2f88aaf8-9e80-45c7-8d6c-2c67dd677dd8.png">

if you type a `\` before any letter in a given command, it's guaranteed bash will get it, bypassing any aliases.

With that in mind, your mission is to find the zip file containing the flag and unzip it. The password can be found hiding in the aliases, specifically under fzip:

`alias fzip='/usr/bin/zip -P "/bin/funzip"'`

There's a problem: the disk is read only, even in /tmp/. So you can't extract the file. One solution would be to copy the file and unzip it locally, but you can just unzip directly to stdout:

<img width="492" alt="image" src="https://user-images.githubusercontent.com/2973929/234147085-05325cf5-af79-4c7e-92ed-89df1ff4c5e5.png">

