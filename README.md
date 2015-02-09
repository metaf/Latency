##Latency and you
###Alternate Title: "F$%^KING LAG"
____

####What is latency?
Pointless waiting.  

####What if computers operated on a human timescale?
Consider a CPU cycle to be roughly equivalent to 1 sec.

>Why one second?

Well, how long does it take you to add 2 numbers?  Probably somewhere between .1 and 10 seconds, so we're within an order of magnitude.

I got some numbers from here: http://blog.codinghorror.com/the-infinite-space-between-words/


| Operation | real latency | scaled latency |
| -------------- | ---------------:|-----------------:|
| 1 cycle | .3 ns | 1 sec |
| L1 Cache access | .9 ns | 3 s |
| L2 Cache access | 2.8 ns | 9s |
| L3 Cache access | 12.9 ns | 43 s|
| RAM Access | 120 ns | 6 minutes|  
| SSD IO | 50-150 microseconds | 2-6 days |
| HDD IO | 1-10 ms | 1-12 months |
| Network: SF to NYC | 40 ms | 4 years |
| Network: SF to AUS | 180 ms | 19 years |

How many numbers can you add together in 19 years?  Something like 31,536,000.

#### Is that so bad?
>My code doesn't do anything like "accessing RAM", I just use *variables*.

Yes, it's bad.  Realistically though, you're not going to write code that avoids using RAM.  This is all magic'd out by processor design to optimize cache use, and compiler design to make code cache-friendly.  Someday you might be one of them!

At a higher level though, you might write code to keep things in RAM rather than disk.  Even more importantly, you might need to consider network latency when deciding how to implement certain applications.

#### Can I have an example?
Yes.

##### For disk vs RAM latency, let's look at a ramFS:
Time to create 4MB of files in RAM: 1.5s

    cd /dev/shm/
    time bash -c "for i in \$(seq 1000); do touch \$i.file; done"

Time to create 4MB of files on rotating disk: 1.5s

    cd /home/taf/data/test/latency
    time bash -c "for i in \$(seq 1000); do touch \$i.file; done"



>###SHIT.  There goes my demo.

It looks like I must have a really good disk, or some really bad RAM.  But actually, linux is **caching** for me.  The disk isn't really done yet, but it goes ahead and tells me it is, because it'll take care of everything for me (unless there's a power outage).

We can see what the numbers really are if we force the disk writes.  One way to do this is to use dd conv=fsync.

Let's run the command below with "conv=fsync", and without it,
on a spinning disk, an SSD, and a ramdisk:

    time bash -c "for i in \$(seq 1000); do dd if=/dev/zero of=\$i.file bs=1 count=1 conv=fsync; done"

|||
|-|-|
| hdd, force sync | 38s |
|hdd, no force sync | 1.8s |
| ram, force sync | 1.8s |
| ram, no force sync | 1.6s |
| ssd, force sync | 7.5s |
| ssd, no force | 1.8s |

##### What about over a network?

Let's take those 1000 files, and copy them to a remote server somewhere, with a ping time of about 100ms.

Time to transfer:
# FOREVER


    time bash -c "for i in \$(seq 1000); do scp \$i.file taf@s0.taf.kim:/home/taf/latency/.; done"

>That sucks.

The truth is in this situation, a good portion of the latency is actually spent establishing the SSH connection.  It's still latency though!

>If latency is so bad, how am I supposed to do *anything* over a network?

This problem isn't actually so bad.  One thing we could do is parallelize it!

    find . -mindepth 1 | xargs -P 8 -n 1 ../scpAFile

That's not *so* bad, but will still take a matter of minutes.

What if we only copied *one* file?

    tar cvf - * | ssh taf@s0.taf.kim "tar xvf - -C /home/taf/latency/"

Time to complete: 17 seconds on a slow link without compression

#TL;DR
Throughput can be limited much more by latency than by bandwidth.
