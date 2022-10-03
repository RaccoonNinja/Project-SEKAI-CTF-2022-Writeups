## Skills required: solving system of linear equations

I wanted to try using [z3](https://pypi.org/project/z3-solver/) for this challenge but I couldn't make it solve fast enough. I also made some careless mistakes here and there.

## Solution:

Here's a quick description of what's happening:
- A truly random secret `secret` is obtained (which is outputed to us)
- From `secret`, a pseudo-random integer stream `A` (of *32(a.k.a. hash length of sha256)\*4=128* integers) is created with hashing (which we can calculate)
- The flag is split into chunks of `128/4=32` bytes, each chunk is then:
  - AND-ed with each of the integers in `A`
  - passed through a lossy function `happiness` before returned to us

```py
def happiness(x: int) -> int:
    return x - sum((x >> i) for i in range(1, x.bit_length()))
```

The `happiness` function really evaluates the number of 1s in the binary expansion. This can be guessed from trying small inputs (which I did), but here's a more formal proof:
1. Consider `happiness(2**n)`, it must always evaluate to 1 (100000... - 11111... = 1)
2. Each bit can be considered separately with the final result as the sum. So the result is the count of `1`s in the binary expansion. QED

This is the key leading to the insight to tackle the challenge by solving linear equations. Using smaller integers as illustration here:

```
bit_size = 4
# flag = 13
A = [9, 15, 11, 5]
# and_result = [9, 13, 9, 5]
enc = [2,3,2,2]

# denoting the ith bit of flag as f{i}, here is the system of equations:
f0+  +  +f3 = 2
f0+f1+f2+f3 = 3
f0+f1+  +f3 = 2
f0+  +f2    = 2
# solving which reveals the flag
```

Since the flag value is not processed in any random way before the operation, more equations can be added if we talk to the server multiple times.

I made a silly mistake of assuming there are 128 variables, resulting in non-integer solutions. Upon double checking I realized that *the bit length of sha-256 was indeed 256* ・ω・

For the technical part of solving a 256-variable linear equation, I used [sagemath](https://www.sagemath.org/download.html). I have more experience with it for crypto challenges involving abstract algebra.

Here is my sloppy sage script:
```sage
from pwn import *
import hashlib, nclib

def gen_pubkey(secret: bytes) -> list:
    def hash(m): return hashlib.sha256(m).digest()
    state = hash(secret)
    pubkey = []
    for _ in range(len(hash(b'0')) * 4):
        pubkey.append(int.from_bytes(state, 'big'))
        state = hash(state)
    return pubkey

BIT_SIZE = 256
secrets = [] # [2 #req][128 #keylength] => [2][128][256]
keys    = [] # [2 #req][3 #frag][128 #keylength]
number_of_fragments = 3

def decompose(x):
  return [int(x&(2**j)>0) for j in range(BIT_SIZE)]

for _ in range(2):
  io = remote('challs.ctf.sekai.team', 3001)
  h = bytes.fromhex(io.recvline().strip().decode('utf-8'))
  secrets.append(gen_pubkey(h))
  keys.append(eval(io.recvline().strip())) # eval
  if not number_of_fragments:
    number_of_fragments = len(keys[-1])
  else:
    assert number_of_fragments == len(keys[-1])
  assert len(secrets[-1]) == len(keys[-1][0])
  io.close()

for i in range(len(secrets)):
  secrets[i] = [decompose(s) for s in secrets[i]]

all_secrets = secrets[0]+secrets[1] # [256,256]
all_keys = [[],[],[]] # [3,256]

for key in keys: #[3][128]
  for j in range(number_of_fragments):
    all_keys[j] += key[j]

# https://webs.um.es/gemadiaz/miwiki/lib/exe/fetch.php?media=matrices_and_gaussian_elimination_--_sage.pdf

for frag_n in range(number_of_fragments):
  A=matrix(QQ, all_secrets)
  b=column_matrix(QQ, all_keys[frag_n])
  Ab=A.augment(b,subdivide=true)
  # print(Ab)
  # print("")
  print(Ab.echelon_form())
  print("")
  # print("")
```

I didn't bother to read back the final column in sage so I tidied up the text and passed the result to [CyberChef](https://cyberchef.org/#recipe=Reverse('Character')From_Binary('Space',8)) to obtain the flag.
