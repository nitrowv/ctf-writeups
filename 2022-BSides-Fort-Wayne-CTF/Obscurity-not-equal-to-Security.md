# Obscurity != Security

**Writeup by:** nitrowv   
**Category:** Reversing   
**Points:** 50

In this challenge, we are given a binary that, when executed, has the following output:

```
┌──(tanner㉿kali)-[~/ctf/bsides]
└─$ ./RE2
Hello BSIDES FW CTF Contestant!!

Can you find my secret?

****MENU****
1 - SysInfo
2 - Search - (WIP)
3 - (WIP)
4 - (WIP)
Please select Menu Number: 
```

Selecting option 1 results in the following;

```
Please select Menu Number: 1
Why did you trust this application?!?!
```

Whereas selecting any of the other options shows the following:

```
Please select Menu Number: 2   
Nothing to see Here... Yet
```

Before using tools such as Ghidra, I like to run the `strings` to see if there is any helpful information that hasn't been obfuscated. When I ran `strings` on this binary, I did find some interesting strings listed:

```
Hello BSIDES FW CTF Contestant!!
Can you find my secret?
****MENU****
1 - SysInfo
2 - Search - (WIP)
3 - (WIP)
4 - (WIP)
Please select Menu Number: 
PLEASE INSERT A VALID OPTION.
YOU FOUND MY SECRET!!!
FFEjwtItjxStyJvzfqSzqqTwItjxNyFF
FLAG:
bsftw{
Why did you trust this application?!?!
Nothing to see Here... Yet
Where do you computers start counting?
Not here... look some where else
basic_string::_M_construct null not valid
;*3$"
```

We can see a flag prefix and a string above it which doesn't really make sense. There is also a helpful hint in the form of "Where do you computers start counting?". Since many programming languages do not start counting at 0 instead of 1, maybe we could try that as an input?

```
┌──(tanner㉿kali)-[~/ctf/bsides]
└─$ ./RE2
Hello BSIDES FW CTF Contestant!!

Can you find my secret?

****MENU****
1 - SysInfo
2 - Search - (WIP)
3 - (WIP)
4 - (WIP)
Please select Menu Number: 0
YOU FOUND MY SECRET!!!
FLAG:bsftw{AAZeroDoesNotEqualNullOrDoesItAA}
```

That did the trick! We find the flag of `bsftw{AAZeroDoesNotEqualNullOrDoesItAA}` by using an option that was not made clear to us by the program. Let it be known that obscurity does not in fact, equal security, as evidenced here.

An interesting thing that I noticed while writing this writeup is that the string we found earlier, `FFEjwtItjxStyJvzfqSzqqTwItjxNyFF`, bears a resemblence to the flag. The FF at the beginning and end of this string seems to match with the AA in the actual flag. Sure enough, running it through CyberChef reveals that when the string is rotated by 21 (or -5) characters, we get the flag.