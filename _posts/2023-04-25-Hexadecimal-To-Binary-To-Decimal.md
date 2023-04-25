---
layout: post
title: Hexadecimal To Binary To Decimal
subtitle: IP & MAC Addresses
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [hex, binary, decimal]
comments: true
---

There are different ways to count. Our system is called, based-ten or decimal. Not sure reason why, but most probably it was convenient for us humans, which have 10 fingers.  
With this system we have numbers: 0,1,2,3,4,5,6,7,8,9 before using an extra digit and "switch" to 10.

Hopefully you are familiar with the following then:  
* 0 - zero
* 1 - one
* 2 - two  
* 3 - three
* 4 - four
* 5 - five
* 6 - six
* 7 - seven
* 8 - eight
* 9 - nine
* 10 - ten
* 11 - eleven
* [...]
* 20 - twenty
* [...]

However, to avoid later confusion please start reading numbers like this:
* [...]
* 10 - one zero
* 11 - one one
* 20 - two zero
* 21 - two one

Now, let us suppose that, we humans, only had 2 fingers on each hand!  

Probably, our numbering system would have been a base four system. In this scenario, we have would be counting as
* 0 - zero
* 1 - one
* 2 - two
* 3 - three
* [Now we run out of fingers and need to use an extra digit ]
* 10 - ONE ZERO (decimal equivalent = 4)
* 11 - ONE ONE (decimal equivalent = 5)
* 12 - ONE TWO (decimal equivalent = 6)
* 13 - ONE THREE (decimal equivalent = 7)
* 20 - TWO ZERO (decimal equivalent = 8)

Great, now let us suppose we have a binary system, or a based 2 system. Where we only have two digits to represent numbers.
  
Say we have two digits "X  X" to represent numbers. If you only have these two digits to represent numbers, then we would only could count up to 2^2 numbers. So, with only two available digits, we could only represent four unique combinations, or four numbers.  
Example:
* **00** - ZERO ZERO (decimal equivalent = 0)
* **01** - ZERO ONE (decimal equivalent = 1)
* **10** - ONE ZERO (decimal equivalent = 2)
* **11** - ONE ONE (decimal equivalent = 3)

There are no more unique combinations giving constrained of only two available digits.

Suppose we add one more available digit to our binary system. Then, we would be able to count up to 2^3. This means that, with this extra digit we could count up to 8 numbers!

Example:
* **000** - ZERO ZERO ZERO (decimal equivalent = 0)
* **001** - ZERO ZERO ONE (decimal equivalent = 1)
* **010** - ZERO ONE ZERO (decimal equivalent = 2)
* **011** - ZERO ONE ONE (decimal equivalent = 3)
* **100** - ONE ZERO ZERO (decimal equivalent = 4)
* **101** - ONE ZERO ONE (decimal equivalent = 5)
* **110** - ONE ONE ZERO (decimal equivalent = 6)
* **111** - ONE ONE ONE(decimal equivalent = 7)

We have exhausted all possible combinations, if we wanted to keep counting we would have to add one more available digit and that would make us able to count up to 2^4.  

Binary is important because this is what computers understand. Computers can only understand bits. A bit is anything that could only take two possible values. In computing this translated to 1 (electricity pulse) and 0 (no pulse).

## IP Address
This is a 32-bit address that identifies a host on a network. It is typically written in dotted-decimal notation. Each number represents an 8 bit number, and this is what the computer understands.

Say we have the following IP address:  
192.168.10.55

This is in decimal, so we understand it. But machines need this in binary. Now, each section correspond to 8 bits, also called an octet.

00000000.00000000.00000000.00000000

Each octet has 8 bits, or 8 available digits. meaning each octet can count from 0 up to 2^8 = 256. As zero is counting as a number, each octet can a a value from 0-255.  
So, we know we can arrange each octet in 256 unique combinations that make it possible to count from 0 to 255.

Need to memorize this table:  
| Base    | 2^7 | 2^6 | 2^5 | 2^4 | 2^3 | 2^2 | 2^1 | 2^0 |
|---------|-----|-----|-----|-----|-----|-----|-----|-----|
| Binary  | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   |
| Decimal | 128 | 64  | 32  | 16  | 8   | 4   | 2   | 1   |


With just the sum of these eight values you can generate any number up to 255!
  
This table means that if we only turn on the last bit we would have a value of 1 decimal value.
  
00000001 = 1 decimal value

If we turn on the third last bit, we would have a value of 4 in decimal.
  
00000100 = 4 decimal value.

Now, if we turn on the first two bits, we would have 128 + 64 in decimal value.
  
11000000 = 192.

So, I think we can resume converting our IP Address from decimal to binary.
192.168.10.55
* **192** = 11000000 - First bit on is 128 + second bit on is 64.
* **168** = 10101000 - First bit on is 128 + third bit on is 32 + fifth bit on is 8.
* **10** = 00001010 - Fifth bit on is 8 + seventh bit on is 2.
* **55** = 00110111 - Third bit on is 32 + fourth bit on is 16 + sixth bit on is 4 + seventh bit on is 4 + last bit on is 1.

IP into decimal: 
11000000.10101000.00001010.00110111
  
Let us see one more example:
10.106.0.41
* **10** = 00001010
* **106** = 01101010
* **0**= 00000000
* **41**= 00101001
  
IP into decimal:
00001010.01101010.00000000.00101001

## Hexadecimal
Hexadecimal is just another number system. It is a base-16 number system. Meaning we have 16 available values.

Here is a table for your reference.

| Hexadecimal | Binary    | Decimal |
|-------------|-----------|---------|
| 0           | 0000      | 0       |
| 1           | 0001      | 1       |
| 2           | 0010      | 2       |
| 3           | 0011      | 3       |
| 4           | 0100      | 4       |
| 5           | 0101      | 5       |
| 6           | 0110      | 6       |
| 7           | 0111      | 7       |
| 8           | 1000      | 8       |
| 9           | 1001      | 9       |
| A           | 1010      | 10      |
| B           | 1011      | 11      |
| C           | 1100      | 12      |
| D           | 1101      | 13      |
| E           | 1110      | 14      |
| F           | 1111      | 15      |
| Extra | Extra | Extra 
| 80          | 1000 0000 | 128     |
| FF          | 1111 1111 | 255     |
| E0          | 1110 0000 | 224     |
| F0          | 1111 0000 | 240     |


## MAC Addresses
MAC addresses are needed to talk to other devices on a network. Each NIC have has a hardcoded MAC address. This accomplishes Hop-To-Hop delivery.
    
Each MAC address is 12 hexadecimal value. Each hexadecimal value corresponds to four binary bits. So, a MAC address is a 48 bit number.  
The first half of a MAC address is the Organizationally Unique Identifier(OUI). The second half is the unique identifier.

So, for a MAC of 00-00-00-00-00-00
  
* Binary:
  * 0000 0000. 0000 0000. 0000 0000. 0000 0000. 0000 0000. 0000 0000
  
Example of MAC Address D8-C0-A7-CD-7B-5F
* Binary
  * **D8**
    * D = 1101
    * 8 = 1000 
    * D8 = 1101 1000 = 216 in decimal.
  * **C0**
    * C = 1100
    * 0 = 0000
    * C0 = 1100 0000 = 192 in decimal.
  * **A7**
    * A = 1010
    * 7 = 0111
    * A7 = 1010 0111 = 167 in decimal.
  * **CD**
    * C = 1100
    * D = 1101
    * CD = 1100 1101 = 205 in decimal.
  * **7B**
    * 7 = 0111
    * B = 1011
    * 7B = 0111 1011 = 123 in decimal.
  * **5F**
    * 5 = 0101
    * F = 1111
    * 5F = 0101 1111 = 95 in decimal.
* All together: 216-192-167-205-123-95

---

Here a list of exercises for you to practice.

Convert Hex to Decimal value
* 6B
* FF
* C0
* 80
* FE

Convert Binary to Decimal:
* 11000111
* 00000011
* 01110001
* 00001010
* 11111111

Convert Decimal to Binary
* 54
* 13
* 192
* 193
* 255