**Category:** Miscellaneous 

**Difficulty:** Easy

For this challenge, we are given a docker instance. When we connect to it using netcat, we get the following response:

```
You must answer 500 questions. You have only a few seconds for each question! Be fast! ⏰

Question 1:

3530980 * 8434446 - 8002443 + 3658989 - 4605602 + 9021196 * 7743078 - 7259765 * 5222477 - 6486189  = ?

Answer: 
```

Well, when we have to answer 500 questions and we don't have a whole lot of time to do so, we have to script it. Initially, I wanted to write a script for this in Bash using something like `bc`. But in the end, I decided to use Python since I'm more comfortable with it.

First things first, we need to establish a connection to the docker instance. I did that by creating a socket to connect to the docker instance.

```py
import socket
import re

if __name__ == '__main__':
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(('161.35.47.235', 30371))
```

Next, we have to make sense of the data that we're given. For each iteration, we take the recieved data 4096 bytes at a time and concatenate it together. Since we know that we're looking for HTB in our final output, we'll decode that and display it if it is there.

```py
    while True:
        data = b''
        while True:
            chunk = client.recv(4096)
            data += chunk
            if len(chunk) < 4096:
                break

        if 'HTB' in data.decode('utf-8'):
            print(data.decode('utf-8'))
            break
```

Now we have to restrict our input to the math problem that we're given. Initially, I wanted to do this by restricting it to only one particular line, but I noticed that the output has a certain amount of characters at the beginning and end of the output. To take advantage of this, I used substrings:

```py
        if i == 1:
            stripper = str(data)
            stripped = stripper[169:-17]
            expression = stripped
        elif i in range (2,10):
            stripper = str(data)
            stripped = stripper[54:-17]
            expression = stripped
        elif i in range (10,100):
            stripper = str(data)
            stripped = stripper[55:-17]
            expression = stripped
        elif i in range (100,501):
            stripper = str(data)
            stripped = stripper[56:-17]
            expression = stripped
        else:
            break      
```

The number of bytes/characters at the beginning of the problem depended on the iteration. The first problem had 169 characters before the expression, problems 2-9 had 54, 10-99 had 55, and 100-500 had 56. The amount of characters after the expression remained constant at 17.

Now that we have our expression, we need to evaluate it and send the result to the server, taking care to use an escape character to handle division!

```py
        if '/' in expression:
            expression = expression.replace('/', '//')

        result = eval(expression)

        print("Result " + str(i) + ": "+ expression + ' = ' + str(result))

        data = str(result).encode('utf-8') + b'\n'
        client.send(data)
        i += 1
```

Our final script looks like this:

```py
#!/usr/bin/python3

import socket
import re

if __name__ == '__main__':
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(('161.35.47.235', 30371))
    i = 1

    while True:
        data = b''
        while True:
            chunk = client.recv(4096)
            data += chunk
            if len(chunk) < 4096:
                break

        if 'HTB' in data.decode('utf-8'):
            print(data.decode('utf-8'))
            break

        if i == 1:
            stripper = str(data)
            stripped = stripper[169:-17]
            expression = stripped
        elif i in range (2,10):
            stripper = str(data)
            stripped = stripper[54:-17]
            expression = stripped
        elif i in range (10,100):
            stripper = str(data)
            stripped = stripper[55:-17]
            expression = stripped
        elif i in range (100,501):
            stripper = str(data)
            stripped = stripper[56:-17]
            expression = stripped
        else:
            break         

        if '/' in expression:
            expression = expression.replace('/', '//')

        result = eval(expression)

        print("Result " + str(i) + ": "+ expression + ' = ' + str(result))

        data = str(result).encode('utf-8') + b'\n'
        client.send(data)
        i += 1
```

And here is the output for our last 6 problems:

```
Result 495: 5829539 * 7891189 - 1852728 + 2768181 * 6477305 - 5407490  = 63932339403858
Result 496: 4263585 + 9263964 + 8596298 - 6154845 - 3743678 * 9527351  = -35667318367976
Result 497: 6408220 + 1535312 - 2247773 + 328130 * 668175 - 4887850  = 219249070659
Result 498: 8237826 - 8879669 + 3862390 - 3707692  = -487145
Result 499: 6096850 * 1970281 + 1726552 * 9428971 - 4458354 * 3766029  = 11501825996576
Result 500: 8190139 - 3322118 + 4577218 * 1580113 * 4819382 + 2161096 + 2681390 - 6518522 - 1080232 - 5055165  = 34856284729963574776

Time: 0.10
Correct! ✔
Congratulations! 🎉
Here is a 🎁 for you: HTB{2_plu5_tw0_1s_4_m1nu5_1_th4t5_3_qu1ck_m4fs!!}
```

Which contains our flag: `HTB{2_plu5_tw0_1s_4_m1nu5_1_th4t5_3_qu1ck_m4fs!!}`