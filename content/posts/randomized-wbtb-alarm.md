---
title: "Randomized WBTB Alarm"
date: 2020-07-05T17:54:45Z
---

This is a quick Python script I wrote to aid with the [CANWILD](https://lucid.fandom.com/wiki/CANWILD
) lucid
dreaming method. The program is an alarm clock with a randomized snooze
function that prevents the sleeper subconsciously learning the snooze
schedule.

The alarms automatically turn themselves off so the sleeper doesn't have to
move after being awakened. Snooze time is fully random, and can be anywhere
from a few seconds to 90 minutes.

It's run from the command line in one of two modes:


1) python sleep.py block

This mode allows four hours of sleep before setting off an initial alarm, then
it begins the randomized snooze schedule. It is designed to be run at the
begining of the night, when first going to bed.


2) python sleep.py wbtb

This mode is for when you wake up for whatever reason, and you want to
immediately go into the randomized snooze schedule. It's basically the first
mode without the initial four hour sleep allowance. "wbtb" stands for
"wake-back-to-bed."


There are two alarm wav files included in the repo. One is a full length alarm
for waking the sleeper after four hours. The other is a truncated alarm to be
played during the snooze schedule.

If you still don't get what this is all about, please read the linked article
above to learn about CANWILD and lucid dreaming in general.

Below is the actual code:

&nbsp;
&nbsp;


``` Python
import scipy.io.wavfile as scw
import sounddevice as sd
from time import sleep
from random import randint
import sys




f_sample_rate, full_alarm = scw.read(
    "full_alarm.wav")

t_sample_rate, truncated_alarm = scw.read(
    "truncated_alarm.wav")




def block():

    sleep(14400)

    sd.play(full_alarm)
    sd.wait()

    while True:

        sleep(
            randint(
                1, 5401))

        sd.play(truncated_alarm)
        sd.wait()




def wbtb():

    while True:

        sleep(
            randint(
                1, 5401))

        sd.play(truncated_alarm)
        sd.wait()




if __name__ == "__main__":

    try:

        mode = sys.argv[1]

        globals()[mode]()

    except Exception as error:

        print(
            "ERROR", 
            "\n" *2, 
            error)






```


&nbsp;
&nbsp;


[Full code repo available on Github](https://github.com/Capybasilisk/Randomized-WBTB-Alarm)

&nbsp;

Related posts:

[Speculative Fuction Bot](https://capybasilisk.com/posts/2020/04/speculative-fiction-bot/)

[EXP-RTL: Exponential Retaliation In Iterated Prisoner's Dilemma Games](https://capybasilisk.com/posts/2020/04/exp-rtl-exponential-retaliation-in-iterated-prisoners-dilemma-games/)

[Interleaved Neighborhood Algorithm: Fully Exploratory Optimization](https://capybasilisk.com/posts/2020/06/interleaved-neighborhood-algorithm-fully-exploratory-optimization/)

&nbsp;

[About Me](https://capybasilisk.com/about/)

