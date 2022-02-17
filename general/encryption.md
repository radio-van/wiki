# Contents

- [symmetric](#symmetric)
- [assymmetric](#assymmetric)
- [signing](#signing)

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

