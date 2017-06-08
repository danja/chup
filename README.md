# Crazy Hutter Play

Somewhere to jot down notes about a general idea I've been noodling around. I have been using pen & paper, but as code might well ensue it makes sense to put things here.

The [Hutter Prize](http://prize.hutter1.net/) is a challenge to compress data *a lot*. Specifically, create a compressed version (self-extracting archive) of the 100MB file enwik8 (a bunch of pages from Wikipedia) of less than about 16MB. I have absolutely no ambition of approaching the challenge, but it offers a good starting point to what I've been thinking about.

I doubt very much that the kind of thing I'm about to describe will be any good for Hutter, but it is fun to ponder. I'm pretty sure it's fairly closely related to some kinds of encryption, but that's not my field at all, so any similarities between this and real work are purely coincidental.

## The General Idea

Begin with a [Pseudorandom number generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator). Probably based on a [Linear-feedback shift register](https://en.wikipedia.org/wiki/Linear-feedback_shift_register) (LFSR). **These things are wonderful!** I first encountered them in the context of music synthesis, as, given a long enough sequence, their output approximates [white noise](https://en.wikipedia.org/wiki/White_noise) in a very predictable fashion. It's how a lot of the early electronic games produced explosion noises.

They only need a handful of gates (or a few lines of code), looking something like this :

![diagram of 16-bit Fibonacci LFSR](https://upload.wikimedia.org/wikipedia/commons/1/16/LFSR-F16.gif "16-bit Fibonacci LFSR")

*A 16-bit Fibonacci LFSR. The feedback tap numbers shown correspond to a primitive polynomial in the table, so the register cycles through the maximum number of 65535 states excluding the all-zeroes state. The state shown, 0xACE1 (hexadecimal) will be followed by 0x5670.*

Now to 'compress' some data, start with the definition of the LFSR and the seed. Run the thing. Then do *extensive* searching of its output for patterns matching sequences of the input data.

Say the data is the string "Hello". The output of a sequence generator, mapped to characters, at it's 100th point might be:

...a p v I R n _ k e **H e l** Q v l g...

There, at position 110 are the first three characters of the data. So record that as a pair *(position in the sequence, number of characters)*, ie (110, 3). Now later in the sequence, say at it's 200th point, there's :

...K  s ! m **l o** s W _ ...

yielding (203, 2).

That's the full 5-character string represented in 4 8-bit numbers! Woo-hoo!

You will need some kind of heuristics for deciding when to stop searching - maybe have lots of searches running in parallel, pick the one that yields the best result within a given max chunk of the sequence). So if a little further on in the sequence you get ...ello... that may be more efficient to store.

Ok, this is pretty bogus as compression. Chances are 'Hel' won't occur so soon, and you end up with pointers that are bigger than the data you are trying to represent. But it kinda works as pseudo-encryption. If the sender and recipient have identical machines (and no-one else does), like [Enigma](https://en.wikipedia.org/wiki/Enigma_machine) the message will be pretty well hidden.

As [stated on Wikipedia](https://en.wikipedia.org/wiki/Hutter_Prize), "The goal of the Hutter Prize is to encourage research in artificial intelligence (AI).". Noting that the machine described above has recursion at it's heart, I don't think I'd be going too far out on a limb by pointing out parallels to [The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/). A
 RNN trained on Wikipedia text could be viewed as a quasi-pseudorandom sequence generator that is optimised for Wikipedia-like text.

## 2am Encoding

I often suffer from insomnia. This kind of stuff doesn't help. Here's what I ended up trying to do in my head last night.

If, rather than the kind of sequence generator described above you just use the sequence of integers expressed in binary, dropping leading zeroes after the first 0, you get :

0 1 10 11 100 101 110 111 1000 1001...

This too can be compressed using a similar system. If you squeeze the bits together:

01101110010111011110001001...

you can point to chunks along the way *(position in the sequence, number of characters)* again :

0 (0,1)  
1 (1, 1)  
2 (2, 2)  
3 (3, 2)  
...

*But*, in the bit-squeezing, 3 could have been expressed more economically with (1, 2)

Continuing to 4 bits, this squeezes to :

0110100111100010101011

4 = 100 (4, 3)  
5 = 101 (3, 3)  
6 = 110 (2, 3)  
7 = 111 (7, 3)  
8 = 1000 (8, 4)  
9 = 1001 (4, 4)  
10 = 1010 (10, 4)  
11 = 1011 (11, 4)  
12 = 1100 (9, 4)  
13 = 1101 (1, 4)  
14 = 1110 (8, 4)  
15 = 1111 (7, 4)

*I'm counting from 0 on position and may well have made a mistake or two there*

There are two numbers for ever one in the original, but this overhead could maybe be reduced by taking into account whatever the patterns are in the values - the second value looks like it follows a very simple sequence.

Ok, I reckon reality prevents this from ever being anything remotely  compression, nor can I see any other practical application right now. But it's more challenging than counting sheep.  

## See Also

Quite often when I get insomnia I end up watching youtube videos of perpetual motion machine makers, Flat Earth & conspiracy theories and almost anything that mentions [Tesla inventions](https://www.youtube.com/results?search_query=tesla+inventions).

But on the Web more generally it can be hard to tell if something radically out there might be a scam, the work of someone delusional or (occasionally) something that contains a seed of a great idea. Related to the above, I recommend - scam, delusion or great idea? -

  * **The Sloot Digital Coding System**, eg see [The Stick of Jan Sloot](http://www.spronck.net/sloot.html) (given the figures quoted and *a fund-raising demo* , I'm inclined towards the first possibility, though there is certainly something of a seed too)
  * [Recursive Digital Management](http://www.recursivedigital.com/) (working prototype in 2012 then no updates? Hmm.)

I'm also trying to figure out where [Chaos Theory](https://en.wikipedia.org/wiki/Chaos_theory) figures in all this, without adding myself to this list. (eg think of the easy-peasy [Logistic Map](https://en.wikipedia.org/wiki/Logistic_map) equation as an *analog* pseudorandom sequence generator...).
