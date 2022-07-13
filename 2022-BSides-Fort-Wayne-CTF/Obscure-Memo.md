# Obscure Memo

**Writeup by:** nitrowv   
**Category:** Steganography  
**Points:** 50

For this challenge, we are given two files; an image named `web_logo.jpg` and a text file named `StegPasswords.txt`, presumably containing passwords to extract data hidden in the image. The appearance of the image is the same as in the previous challenge, and the text file contains 101 possible passwords, with one on each line:

```
BSIDES
Wayne
security
Schedule
...
...
```

Since there are this many possible passwords, it isn't feasible to try each of them manually. Fortunately, we can automate this process through scripting!

The actual extraction will be done using steghide, a program that allows for files to be embedded in and extracted from image and audio files.

To extract embedded content from a file, we need to specify what file has what we're looking for (the stego file) and the password/passphrase. This is done using `steghide extract -sf FILE.png -p PASSWORD`.

As for automating this process, it is quite simple. All that's needed is a for loop. My script and its output are fairly crude, but it gets the job done.

```bash
for i in $(cat StegPasswords.txt);
do echo 'Attempting passphrase: ' $i;
steghide extract -sf web_logo.jpg -p $i;
done
```

This script iterates through the password file and uses that line's value in the steghide command.

```
┌──(tanner㉿kali)-[~/ctf/bsides]
└─$ ./steghidebf.sh
Attempting passphrase:  BSIDES
steghide: could not extract any data with that passphrase!
Attempting passphrase:  Wayne
steghide: could not extract any data with that passphrase!
Attempting passphrase:  security
steghide: could not extract any data with that passphrase!
Attempting passphrase:  Schedule
steghide: could not extract any data with that passphrase!

...
...

Attempting passphrase:  BSFTW
wrote extracted data to "BsidesCTF.txt".

...
```

The passphrase that was able to extract the data was `BSFTW`, and steghide wrote the data to a text file named `BsidesCTF.txt`, so let's take a look at that.

```
Hello Bsides CTF Challeneger!
Are you enjoying Bsides FW? I hope you are! There is a flag some where in the text below! Will you find it?

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Aliquam malesuada bibendum arcu vitae elementum curabitur vitae nunc. Massa vitae tortor condimentum lacinia quis vel eros. Accumsan in nisl nisi scelerisque. Morbi quis commodo odio bsftw{This Is Not The Flag You Have Been Looking for!!} aenean sed adipiscing.
...
```

The text file contains lorem ipsum and at least one false flag. With the help of grep and Kali's zsh syntax highlighting, I was able to better see the three occurrences of the flag prefix:

```
... Morbi quis commodo odio bsftw{This Is Not The Flag You Have Been Looking for!!} aenean sed adipiscing...

... Felis imperdiet bsftw{Maybe You Should read the requirements for a flag?} proin fermentum leo vel orci porta...

... Sagittis purus sit amet volutpat consequat bsftw{RegexIsCoolButIsVeryScaryAndHard} mauris nunc congue nisi. Senectus et netus et malesuada fames ac turpis egestas...
```

Looking at the flag format that we've seen so far, only the last fits the format, as it does not contain any spaces. It should be noted that the organizers did provide a regular expression of the flag format on the CTF scoreboard, which would have made this challenge a bit easier had I used it. Either way, we find that the flag is: `bsftw{RegexIsCoolButIsVeryScaryAndHard}`.