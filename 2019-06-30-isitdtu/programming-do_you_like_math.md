# Programming - Do you like math?

We were presented with a server printing out basic math operations between two numbers and asking for the result. The challenge, however, came from the fact that the server pretty printed everything as it was a calculator, so we had to parse the output and then send back the result.
<br>
I decided to tackle this challenge as an opportunity to learn Python, as I barely ever used the language and am not yet familiar with its common functions and syntax.

---

## Problem Output Samples
```
 #####        #       #######          
#     #       #    #  #                
      #       #    #  #          #####
 #####  ##### #    #  ######           
      #       #######       #    #####
#     #            #  #     #          
 #####             #   #####           
```
```
 #####   #####           #####           
#     # #     #  #   #  #     #          
      #       #   # #         #    #####
 #####   #####  #######  #####           
      #       #   # #         #    #####
#     # #     #  #   #  #     #          
 #####   #####           #####           
```

## Constraints
Seeing the server output, some questions arise. Can the length of both numbers be 1 and 2? What operations does it ask us to do?

I've sent several requests to the server and verified that:
- Numbers from 1 to 99 are used on both sides of the operation. So, the output can contain 2, 3 or 4 numbers.
- The operations it outputs are add, sub and mul.
- Some numbers are not as wide as others, same for operation signs.
- Every character is separated by a column of spaces. Equal signs are separated by more than one column of spaces.
- The number of iterations it does is unknown, and it times out after ~ 1-2 seconds if there's no input sent back.

### Tackling the problem
I started by storing line by line (excluding empty lines) on an array. Then, I check what position has the first completely empty column, as that position is the separator between characters. I tag it as a barrier, and then concatenate everything on the left side to a single string, line by line, while replacing # for 1s and empty characters for 0s.
I then pinged the server while printing the first character in the form of our converted string until I had the string for every character.

```
'0' -> "0011100010001010000011000001100000101000100011100"
'1' -> "00100011001010000100001000010011111"
'2' -> "0111110100000100000010111110100000010000001111111"
'3' -> "0111110100000100000010111110000000110000010111110"
'4' -> "1000000100001010000101000010111111100000100000010"
'5' -> "1111111100000010000001111110000000110000010111110"
'6' -> "0111110100000110000001111110100000110000010111110"
'7' -> "1111111100001000001000001000001000000100000010000"
'8' -> "0111110100000110000010111110100000110000010111110"
'9' -> "0111110100000110000010111111000000110000010111110"
```

After doing the conversion from string to the actual character, the character is concatenated to our result string, the current character is trimmed from the strings on the array and the cycle starts over until we have no characters left.
After wrapping the code in a cycle, I also got the strings for the operation symbols as it parsed the other characters in the operation beyond the first.

```
'+' -> "00000001000010011111001000010000000"
'-' -> "00000000000000011111000000000000000"
'*' -> "0000000010001000101001111111001010001000100000000"
'=' -> "00000000001111100000111110000000000"
```

After having the operation converted from the pretty output to a string with the actual values, `eval` can be applied directly and it immediately calculates the result. The result is then sent with `sendline`.

```
#####   #####         #####   #####           
#     # #     #       #     # #     #          
     # #     #             #       #    #####
#####   ###### #####  #####   #####           
     #       #             #       #    #####
#     # #     #       #     # #     #          
#####   #####         #####   #####           

Converted: "39-33"
Eval: 6
```

Finally, I just wrapped everything in another cycle and let it run until it crashed. I found out it does exactly 100 iterations. Swapped the `while True:` cycle for a `for iteration in xrange(100):`.
After that is done, I print every received byte in a loop until we reach `EOF` from the server. I first had tried doing it with `recvline` to try to exit gracefully without forcing an `EOFError` crash, but it was not working as intended, so I just went with the recv() loop.

## GG
`Good job, this is your flag:   ISITDTU{sub5cr1b3_b4_t4n_vl0g_4nd_p3wd13p13}`

## Script
*Python2 and PwnTools required to run*
```
from pwn import *

r = remote("104.154.120.223", 8083)

for iteration in xrange(100):
    print "i = " + str(iteration)

    # Store output
    input = []
    r.recvline() # Empty line
    for i in range(0, 7):
        input.append(r.recvline())
    r.recvline() # Empty line

    # Display output
    for i in range(0, 7):
        print(input[i][:-1])

    # Final operation for eval
    operation = ''

    # Iterate character by character
    while True:
        barrier = 0
        for i in range(0, 9):
            for j in range(0, 7):
                if input[j][i] != ' ':
                    barrier = 1
            if barrier == 0:
                barrier = i
                break
            else:
                barrier = 0
                continue

        s = ''
        for j in range(0, 7):
            for i in range(0, barrier):
                s += input[j][i].replace(' ', '0').replace('#', '1').rstrip()

        # Note: Challenge only appears to have add, sub, mul
        if s == "0011100010001010000011000001100000101000100011100":
            operation += '0'
        elif s == "00100011001010000100001000010011111":
            operation += '1'
        elif s == "0111110100000100000010111110100000010000001111111":
            operation += '2'
        elif s == "0111110100000100000010111110000000110000010111110":
            operation += '3'
        elif s == "1000000100001010000101000010111111100000100000010":
            operation += '4'
        elif s == "1111111100000010000001111110000000110000010111110":
            operation += '5'
        elif s == "0111110100000110000001111110100000110000010111110":
            operation += '6'
        elif s == "1111111100001000001000001000001000000100000010000":
            operation += '7'
        elif s == "0111110100000110000010111110100000110000010111110":
            operation += '8'
        elif s == "0111110100000110000010111111000000110000010111110":
            operation += '9'
        elif s == "00000001000010011111001000010000000":
            operation += '+'
        elif s == "00000000000000011111000000000000000":
            operation += '-'
        elif s == "0000000010001000101001111111001010001000100000000":
            operation += '*'
        elif s == "00000000001111100000111110000000000":
            # End of operation
            break
        elif s == '':
            # Caused by double spacing between operation and equal sign
            pass
        else:
            print "Unhandled symbol! \n"
            print s

        # Remove current character from input
        for i in range(0, 7):
            input[i] = input[i][barrier+1:]

    print operation
    print eval(operation)
    print '\n'
    r.sendline(str(eval(operation)))


# Print flag
while True:
    print r.recv()
```
