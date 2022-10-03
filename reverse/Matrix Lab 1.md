## Skills involved: Reversing Java

This is quite easy given the availability of reversing tools online.

## Solution:

We are given a .class Java file. Looking up `java class decompiler online` gives this [very useful tool](http://www.javadecompilers.com/).

I have previous experience with this platform, so I know choosing the second option will give very good outputs and it is usually fast.

```java
public class Sekai {
    private static int length = 6;

    public static void main(String[] arrstring) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter the flag: ");
        String string = scanner.next();
        if (string.length() != 43) {
            System.out.println("Oops, wrong flag!");
            return;
        }
        String string2 = string.substring(0, 6);
        String string3 = string.substring(6, string.length() - 1);
        String string4 = string.substring(string.length() - 1);
        if (string2.equals("SEKAI{") && string4.equals("}")) {
            assert (string3.length() == 6 * 6);
            if (Sekai.solve((String)string3)) {
                System.out.println("Congratulations, you got the flag!");
            } else {
                System.out.println("Oops, wrong flag!");
            }
        } else {
            System.out.println("Oops, wrong flag!");
        }
    }

    public static String encrypt(char[] arrc, int n) {
        int n2;
        char[] arrc2 = new char[12];
        int n3 = 5;
        int n4 = 6;
        for (n2 = 0; n2 < 6 * 2; ++n2) {
            arrc2[n2] = arrc[n3--];
            arrc2[n2 + 1] = arrc[n4++];
            ++n2;
        }
        n2 = 0;
        while (n2 < 6 * 2) {
            int n5 = n2++;
            arrc2[n5] = (char)(arrc2[n5] ^ (char)n);
        }
        return String.valueOf(arrc2);
    }

    public static char[] getArray(char[][] arrc, int n, int n2) {
        int n3;
        char[] arrc2 = new char[6 * 2];
        int n4 = 0;
        for (n3 = 0; n3 < 6; ++n3) {
            arrc2[n4] = arrc[n][n3];
            ++n4;
        }
        for (n3 = 0; n3 < 6; ++n3) {
            arrc2[n4] = arrc[n2][6 - 1 - n3];
            ++n4;
        }
        return arrc2;
    }

    public static char[][] transform(char[] arrc, int n) { // 36 letters to 6*6 grid
        char[][] arrc2 = new char[n][n];
        for (int i = 0; i < n * n; ++i) {
            arrc2[i / n][i % n] = arrc[i];
        }
        return arrc2;
    }

    public static boolean solve(String string) {
        char[][] arrc = Sekai.transform(string.toCharArray(), 6);
        for (int i = 0; i <= 6 / 2; ++i) {
            for (int j = 0; j < 6 - 2 * i - 1; ++j) {
                char c = arrc[i][i + j];
                arrc[i][i + j] = arrc[6 - 1 - i - j][i];
                arrc[6 - 1 - i - j][i] = arrc[6 - 1 - i][6 - 1 - i - j];
                arrc[6 - 1 - i][6 - 1 - i - j] = arrc[i + j][6 - 1 - i];
                arrc[i + j][6 - 1 - i] = c;
            }
        }
        return "oz]{R]3l]]B#50es6O4tL23Etr3c10_F4TD2".equals(
            Sekai.encrypt(Sekai.getArray(arrc, 0, 5), 2) +
            Sekai.encrypt(Sekai.getArray(arrc, 1, 4), 1) +
            Sekai.encrypt(Sekai.getArray(arrc, 2, 3), 0);
    }
}
```

Now it is just a basic crypto challenge.

My approach is accessible enough for beginners to Java ~like me~: just use an [online Java playground](https://www.online-java.com/) and print the intermediate output.
I used `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ` as the flag body so the intermediate states will be unambiguous.

Here is an overflow of what it does:
1. Checks if the string is of the correct format `SEKAI{...}` with 36 characters between the curly braces. (*Sekai.main*)
1. Places the flag body into a 6\*6 grid. (*Sekai.transform*)
1. Permute the grid entries (It really is a 90 degree clockwise rotation) (*Sekai.solve*)
1. Extract the 0th and 5th row and concatenate the former with the reversed latter row content. (Similar for `1,4` and `2,3`, *Sekai.solve* + *Sekai.getArray*)
   1. Permute the combined row content again and XORing them with the 2nd argument of Sekai.encrypt
1. The three parts are finally concatenated and checked against the encrypted string.

The steps can be easily reversed even manually. I only used Python for the XOR step.
