## Skills involved: Reversing exe created by PyInstaller 

This challenge is clear about what to do. Some research is needed for the exact tools.

## Solution:

We are given an exe so maybe it's a good idea to move it to a Windows machine:

![image](https://user-images.githubusercontent.com/114584910/193560239-c58d67d6-f55d-49fc-962d-fd90d5f1dc72.png)

The fact that this exe was written in Python can be either found by the method above, or by `strings` command:

![image](https://user-images.githubusercontent.com/114584910/193560620-581e6d02-cd5b-412c-a1bd-fdf23d56a196.png)

Upon basic [research](https://www.geeksforgeeks.org/convert-python-script-to-exe-file/) we can guess that `pyinstaller` is used for packing the Python code as an .exe. This can be further confirmed by grepping the strings output:

![image](https://user-images.githubusercontent.com/114584910/193561266-db05b4b9-d532-4ba8-9f04-6a6b07a6b52a.png)

Again upon basic research I arrived at [this pyinstall .pyc extractor](https://github.com/extremecoders-re/pyinstxtractor). It is very easy to use.

![image](https://user-images.githubusercontent.com/114584910/193561966-2b4994e7-de55-427b-a5aa-bc6d43abf853.png)

We only need the `Matrix_Lab.pyc` file, which we can pass to another very useful site: https://www.decompiler.com/

```py
# uncompyle6 version 3.7.4
# Python bytecode 3.7 (3394)
# Decompiled from: Python 2.7.17 (default, Sep 30 2020, 13:38:04) 
# [GCC 7.5.0]
# Warning: this version of Python has problems handling the Python 3 "byte" type in constants properly.

# Embedded file name: Matrix_Lab.py
print('Welcome to Matrix Lab 2! Hope you enjoy the journey.')
print('Lab initializing...')
try:
    import matlab.engine
    engine = matlab.engine.start_matlab()
    flag = input('Enter the lab passcode: ').strip()
    outcome = False
    if len(flag) == 23 and flag[:6] == 'SEKAI{' and flag[-1:] == '}':
        A = [ord(i) ^ 42 for i in flag[6:-1]]
        B = matlab.double([A[i:i + 4] for i in range(0, len(A), 4)])
        X = [list(map(int, i)) for i in engine.magic(4)]
        Y = [list(map(int, i)) for i in engine.pascal(4)]
        C = [[None for _ in range(len(X))] for _ in range(len(X))]
        for i in range(len(X)):
            for j in range(len(X[i])):
                C[i][j] = X[i][j] + Y[i][j]

        C = matlab.double(C)
        if engine.mtimes(C, engine.rot90(engine.transpose(B), 1337)) == matlab.double([[2094, 2962, 1014, 2102], [2172, 3955, 1174, 3266], [3186, 4188, 1462, 3936], [3583, 5995, 1859, 5150]]):
            outcome = True
    elif outcome:
        print('Access Granted! Your input is the flag.')
    else:
        print('Access Denied! Your flag: SADGE{aHR0cHM6Ly95b3V0dS5iZS9kUXc0dzlXZ1hjUQ==}')
except:
    print('Unknown error. Maybe you are running the lab in an unsupported environment...')
    print('Your flag: SADGE{ovg.yl/2M6pWQB}')
```

For the technical part, I used the online matlab engine to understand the various operations. It's possible to use sage or numpy while reading matlab's documentation but I figured that it's faster that way, also I can't really remember those steps.

Finally XORing all values with 42 gives the final flag body.
