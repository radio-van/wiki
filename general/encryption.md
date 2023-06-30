# Contents

- [symmetric](#symmetric)
- [assymmetric](#assymmetric)
- [signing](#signing)
- [password entropy](#password-entropy)

# symmetric
One key is used to *encrypt* and *decrypt* message.
Fast.


# assymmetric
There are two keys:
- open (public)
- secret (private)

Message is *encrypted* with **public** key and can be *decrypted* only with secret **private** key.  
Slow.


# signing
Consists of several parts:
- hashing the content of a message to provide integrity (edited content will give another hash)
- encrypting **hash** with **private** key to provide identity (only key owner can encrypt a message)

The result is **signature** which can be *decrypted* with owner's **public** key.  
Decrypted data contains **hash** and some **identity** information. Together with **identification number** they comprise **certificate**.
As only owner can encrypt this data into signature, receiver can be sure that teh message was not changed and was created by 

To grant that **certificate** owner is the same real person, two approaches exist:
- by *Authority center* (`X.509` standart) 
- by *Web of trust* (`PGP` standart)


# password entropy
* 64-bit key in 4 years 9 months
* 72-bit key using current hardware will take about 124.8 years
* Due to currently understood limitations from fundamental physics,
  there is no expectation that any digital computer (or combination)
  will be capable of breaking 256-bit encryption via a brute-force attack.


| Desired password entropy   |             | 8 bits | 32 bits | 40 bits | 64 bits | 80 bits | 96 bits | 128 bits | 160 bits | 192 bits | 224 bits | 256 bits |
| -------------------------- | ----------- | ------ | ------- | ------- | ------- | ------- | ------- | -------- | -------- | -------- | -------- | -------- |
| Arabic numerals            |             | 3      | 10      | 13      | 20      | 25      | 29      | 39       | 49       | 58       | 68       | 78       |
| Hex                        |             | 2      | 8       | 10      | 16      | 20      | 24      | 32       | 40       | 48       | 56       | 64       |
| Case insens                | alpha       | 2      | 7       | 9       | 14      | 18      | 21      | 28       | 35       | 41       | 48       | 55       |
|                            | alphanum    | 2      | 7       | 8       | 13      | 16      | 19      | 25       | 31       | 38       | 44       | 50       |
| Case sens                  | alpha       | 2      | 6       | 8       | 12      | 15      | 17      | 23       | 29       | 34       | 40       | 45       |
|                            | alphanum    | 2      | 6       | 7       | 11      | 14      | 17      | 22       | 27       | 33       | 38       | 43       |
| All ASCII                  | printable   | 2      | 5       | 7       | 10      | 13      | 15      | 20       | 25       | 30       | 35       | 39       |
| All Extended ASCII         |             | 2      | 5       | 6       | 9       | 11      | 13      | 17       | 21       | 25       | 29       | 33       |
| Diceware word list         |             | 1      | 3       | 4       | 5       | 7       | 8       | 10       | 13       | 15       | 18       | 20       |
