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

This one is quite simple, but I'd like to register my solution because it's something just a few people know.

After snooping around, you quickly realize that you can't input almost any command. If you're in the game long enough, you'll know that there are a lot of aliases stopping you

<img width="454" alt="image" src="https://user-images.githubusercontent.com/2973929/234145852-800a2628-86f7-47fe-ace5-19e962430f02.png">

You could just "unalias" each one of them and get the flag, but take a look:

<img width="149" alt="image" src="https://user-images.githubusercontent.com/2973929/234146129-2f88aaf8-9e80-45c7-8d6c-2c67dd677dd8.png">

if you type a `\` before any letter in a given command, it's guaranteed bash will get it, bypassing any aliases.

Then all you have to do is to find the zip file with the flag and unzip it. The password is in the aliases, under `fzip`:

`alias fzip='/usr/bin/zip -P "/bin/funzip"'`

easy:
<img width="492" alt="image" src="https://user-images.githubusercontent.com/2973929/234147085-05325cf5-af79-4c7e-92ed-89df1ff4c5e5.png">

