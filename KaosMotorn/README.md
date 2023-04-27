# Kaos Motorn
## Description
What?

Is?

This?

[Kaos](https://docs.google.com/spreadsheets/d/1yxWyraRKss6Wqbw_ejuws6v92vwdE1AEAP1Cc8oec7M/edit#gid=0)?

<img width="434" alt="image" src="https://user-images.githubusercontent.com/2973929/234678860-11fd5d26-1628-41b4-aea1-487bb23a29ae.png">


### hint
Inputs are in the range 0..9.

## Solution

It's a complex spreadsheet where you could change values in some cells and the gray cells in the middle would change depending on them.

What I did was to get the formula for gray each cell, in order, and reverse their formulas in (yet another) bash script.

```bash
#!/bin/bash

declare -a str
ascii(){
  printf '%x' $1 | xxd -r -p
}


for (( e2=9; e2>=0; e2-- )); do
  for (( b7=9; b7>=0; b7-- )); do
    for (( d14=9; d14>=0; d14-- )); do
      for (( g14=9; g14>=0; g14-- )); do
        for (( j6=9; j6>=0; j6-- )); do
          j13=$(( (d14+b7*e2) % 64 ))
          d11=$(( (e2*g14) % 64 ))
          f12=$(( (e2*b7+d14) % 64 ))
          i10=$(( (j6*g14+b7) % 64 ))
          b13=$(( (b7*j6*g14+5) % 64 ))
          h3=$(( (b7*j6*7) % 64 ))
          g6=$(( (g14*b7+d14) % 64 ))
          f4=$(( (e2*g14+d14+j6) % 64 ))
          c3=$(( (j6+b7+34+g14) % 64 ))
          d5=$(( (e2+j6+b7+d14+g14) % 64 ))

          b5=$(((j13+d11+f12+i10) % 64 ))
          c12=$(( (b13+f12+i10) % 64 ))
          c6=$(( (h3+g6+b13+3) % 64 ))
          e6=$(( (f4+d11+i10) % 64 ))
          f10=$(( (j13+f12+d11+f4+17) % 64 ))
          h11=$(( (c3+h3+f12) % 64 ))
          h13=$(( (h3+g6+f12+d11+b13+b13) % 64 ))
          i5=$(( (h3+d5) % 64 ))
          i7=$(( (f12+d11*g6+h3) % 64 ))
          j3=$(( (f12+d11+d5+f4) % 64 ))

          str[0]=$(ascii $((52+$b5)))
          [ "${str[0]}" != "h" ] && continue
          str[1]=$(ascii $((44+$i7)))
          [ "${str[1]}" != "e" ] && continue
          str[2]=$(ascii $((48+$j3)))
          [ "${str[2]}" != "2" ] && continue
          str[3]=$(ascii $((45+$e6)))
          [ "${str[3]}" != "0" ] && continue
          str[4]=$(ascii $((42+$i5)))
          [ "${str[4]}" != "2" ] && continue
          str[5]=$(ascii $((63-$f10)))
          [ "${str[5]}" != "3" ] && continue
          str[6]=$(ascii $(($h13+93)))
          str[7]=$(ascii $(($c12+68)))
          str[8]=$(ascii $(($h13+74)))
          str[9]=$(ascii $(($i7-5)))
          str[10]=$(ascii $(($c6*6+2)))
          str[11]=$(ascii $(($i7+$b5+$j6-34)))
          str[12]=$(ascii $((91-$c12)))
          str[13]=$(ascii $(($i7+$h11-10)))
          str[14]=$(ascii $(($b5-4)))
          str[15]=$(ascii $(($h11+$h13+$i5+$j3)))
          str[16]=$(ascii $(($h13+$e6)))
          str[17]=$(ascii $(($e6*$h11-25)))
          echo "e2: $e2 b7: $b7 d14: $d14 g14: $g14 j6: $j6"
          echo "${str[@]}" | tr -d ' '
          exit
        done
      done
    done
  done
done
```
Running it:

```bash
$ time ./bruteSheet.sh
e2: 8 b7: 2 d14: 1 g14: 5 j6: 8
he2023{Th4tSKa0Z!}

real    0m15.038s
user    0m18.791s
sys     0m2.192s
```

Checking it in the spreadsheet:

<img width="403" alt="image" src="https://user-images.githubusercontent.com/2973929/234678965-ee3d2cc7-c4af-411f-b539-9f3a9368b6d1.png">

