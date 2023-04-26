# Challenge

A warm, sunny day - perfect weather for a picnic! But what's that - did the bunnies really bring the nice quilt from the living room as a blanket?


### Hint

Cut a beautiful quilt like this into pieces? A shame, but maybe necessary!



## Solution
This one was quite easy and I'm just writing this because I guess it was intended to be much more difficult (given the hint the author gave us).

It's just a huge file full of QR codes and he says maybe we should split the file.

But we won't need to:

```bash
➜  zbarimg quilt.png | cut -f2 -d:| tr -d '\n'
scanned 700 barcode symbols from 1 images in 0.55 seconds

?gniog uoy era erehw ,yeH ?eikooc a ekil uoy dluoW ?eikooc a tuoba woH .noos os evael ton od ,esaelP .meht fo lla evol I dna ,krow fo tol a era yehT !stliuq ym gnitaicerppa rotisiv a evah ot ecin os si ti tub ,gnilbmar ma I .yrroS .ti eb dluohs taht ,haeY }!gnixaler_3tiuq_si_gn1tl1uQ{3202eh ereh thgir ,ti si siht erus ylriaf ma I tuB !yrros ..oooN .}tsafkaerb_rof_gge_siht_deen_I{3202eh ?ebyam sihT ?ti eveileb uoy nac ,eno dlo na saw tahT .ti ton si taht ,yrros ,oN ...ht_si_siht{3202eh ereh ,hA ?ti tup I did erehW .era uoy teb I ?thgir ,gge na rof ereh era uoY !sthguoht eldi ni tsol gnitteg em ta kool tub ,ym ho ym ..ytterp os era yehT !od I erus ytterp ma I ...lleW ?stliuq evol uoy oD !olleH

➜  zbarimg quilt.png | cut -f2 -d:| tr -d '\n'|rev
scanned 700 barcode symbols from 1 images in 0.54 seconds

Hello! Do you love quilts? Well... I am pretty sure I do! They are so pretty.. my oh my, but look at me getting lost in idle thoughts! You are here for an egg, right? I bet you are. Where did I put it? Ah, here he2023{this_is_th... No, sorry, that is not it. That was an old one, can you believe it? This maybe? he2023{I_need_this_egg_for_breakfast}. Nooo.. sorry! But I am fairly sure this is it, right here he2023{Qu1lt1ng_is_quit3_relaxing!} Yeah, that should be it. Sorry. I am rambling, but it is so nice to have a visitor appreciating my quilts! They are a lot of work, and I love all of them. Please, do not leave so soon. How about a cookie? Would you like a cookie? Hey, where are you going?
```
The flag is right there in the middle of the text:

he2023{Qu1lt1ng_is_quit3_relaxing!}
