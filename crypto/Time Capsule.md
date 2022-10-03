## Skills required: simple reversing and Python

Basic crypto challenges often have overlaps with reverse engineering challenges (without the engineering part). This challenge is simple enough.

## Solution:

For basic challenges like this, reversing the procedure usually suffices. The encryption is carried in 2 parts so we can start tackling the second part.

It is a basic XOR with pseudo-random key based on time, with the time value `now` encoded by XORing with 0x42. This can be trivially retrieved:

![image](https://user-images.githubusercontent.com/114584910/193501675-48d04d12-f28c-4037-a3c6-6ef0d838145e.png)

Then the second step is reversed as followings: (remember that `88` is a part of `\x88`)

![image](https://user-images.githubusercontent.com/114584910/193503301-cfccd186-c5b3-4dc8-bb68-8fba1453fa92.png)

If you have peeked into the first part of the encryption, you will know that we are getting somewhere - all the characters are ASCII and the crucial `SEKAI{}` is in it.

We basic debugging techniques we can find out the first part is a simple permutation cipher:

![image](https://user-images.githubusercontent.com/114584910/193500811-396bd63b-0a58-4e2b-a4bb-5a225cf09a50.png)

It splits the message into 8 columns, and concatenate the column contents by an order determined by the true random numbers (`os.urandom`). Unlike the insecure pseudo-random counterpart, we can't know what they are. But since there are only `8!=40320` possibilities, this can be brute-forced easily. Here is the final part of the code:

```py
from itertools import permutations   # itertools is very handy
def decrypt_stage_one(message, key): # permutation cipher
    res = [None]*len(message)
    k = 0
    for i in key: # switch columns and read columns
        for j in range(i, len(message), len(key)):
            # print(j, message[j])
            res[j] = message[k]
            k += 1
    return bytes(res)
    
for perm in permutations(range(8)):
    H = h
    for _ in range(42):
        H = decrypt_stage_one(H, perm)
    if H[:6] == b"SEKAI{":
        print(H)
```

P.S. It is common to see challenges converting the `time.time()` value to int and using it as seed, allowing for potential brute-forcing. It is not very possible in this challenge though.
