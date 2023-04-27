# Code Locked

<img width="304" alt="image" src="https://user-images.githubusercontent.com/2973929/234139203-306f2b32-d408-4128-a8ce-64b023a2d7a4.png">

## Description
Open the code lock at http://ch.hackyeaster.com:2311 to get your 🚩 flag.

Note: The service is restarted every hour at x:00.

## Solution

The challenge greets us with a safe lock. After examining the code, we find out that the lock code has 8 digits and pressing # submits our guess, which is then checked by the checkWASM function.

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

After some tinkering, I came up with a plan to crack the lock by brute-forcing it using the browser's JavaScript console. I used this function to do the trick:

```javascript
function checkLock() {
  const combinations = 100000000;
  for (let i = 0; i < combinations; i++) {
    const combination = i.toString().padStart(8, '0');
    const result = checkWASM(combination);
    if (result !== "You did not open the lock!") {
      console.log("Senha:", combination);
      console.log(result);
      break;
    }
  }
}
```
<img width="457" alt="image" src="https://user-images.githubusercontent.com/2973929/234874086-e8c74d0c-ec90-4d90-a0a3-23e393b3581d.png">


Voilà!

