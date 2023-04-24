<img width="304" alt="image" src="https://user-images.githubusercontent.com/2973929/234139203-306f2b32-d408-4128-a8ce-64b023a2d7a4.png">

# Description
Open the code lock at http://ch.hackyeaster.com:2311 to get your 🚩 flag.

Note: The service is restarted every hour at x:00.

# Solution

The challenge presents a safe lock. After a quick reading of the code, we discover that the code should have 8 digits, and that you should press `#` to submit your guess, which will be checked by the checkWASM function.
```javascript
function press(input) {
    if (input == "*") {
        play("delete.mp3");
        $("#yellow").show(0).delay(200).hide(0);
        code = "";
    } else if (input == "#") {
        msg = checkWASM(code);
        if (msg.startsWith("he2023")) {
            play("success.mp3");
            audioSuccess.play();
            $("#green").show(0).delay(5000).hide(0);
        } else {
            play("fail.mp3");
            $("#red").show(0).delay(1000).hide(0);
        }
        setTimeout(function() {alert(msg);}, 200)
    } else {
        $("#yellow").show(0).delay(200).hide(0);
        play("click.wav");
        code = (code + input).substr(-8, 8);
    }
}
```

After a few minutes trying, the solution I came up with was to brute force the lock using the javascript console of my browser using the following function:
```javascript
function checkLock() {
  const combinations = 100000000;
  for (let i = 0; i < combinations; i++) {
    const combination = i.toString().padStart(8, '0');
    const result = checkWASM(combination);
    if (result !== "You did not open the lock!") {
      console.log("Senha:", combination);
      break;
    }
  }
}
```
<img width="291" alt="image" src="https://user-images.githubusercontent.com/2973929/234139821-36067129-b940-4169-9780-35d05761870e.png">

Then all you had to do was to input this password in the lock using your mouse and hit `#`


