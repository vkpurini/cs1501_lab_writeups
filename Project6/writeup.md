# Project 6 Writeup

**Author:** Vivek Purini

---

## Variable-Width LZW Compression

## Solution

```java
public class LZW {
    private static final int R = 256;    // number of input chars
    private static final int MINW = 9;   // minimum codeword width
    private static final int MAXW = 16;  // maximum codeword width

    public static void compress() {
        int W = MINW;
        int L = 1 << W;

        String input = BinaryStdIn.readString();
        TST<Integer> st = new TST<Integer>();
        for (int i = 0; i < R; i++)
            st.put("" + (char) i, i);
        int code = R + 1;

        while (input.length() > 0) {
            String s = st.longestPrefixOf(input);
            BinaryStdOut.write(st.get(s), W);
            int t = s.length();
            if (t < input.length() && code < L)
                st.put(input.substring(0, t + 1), code++);
            if (code == L && W < MAXW) {
                W++;
                L = 1 << W;
            }
            input = input.substring(t);
        }
        BinaryStdOut.write(R, W);
        BinaryStdOut.close();
    }

    public static void expand() {
        int W = MINW;
        int L = 1 << W;
        int MAXL = 1 << MAXW;

        String[] st = new String[MAXL];
        int i;

        for (i = 0; i < R; i++)
            st[i] = "" + (char) i;
        st[i++] = "";

        int codeword = BinaryStdIn.readInt(W);
        if (codeword == R)
            return;
        String val = st[codeword];

        while (true) {
            BinaryStdOut.write(val);
            codeword = BinaryStdIn.readInt(W);
            if (codeword == R)
                break;
            String s = st[codeword];
            if (i == codeword)
                s = val + val.charAt(0);
            if (i < L)
                st[i++] = val + s.charAt(0);
            if (i == L && W < MAXW) {
                W++;
                L = 1 << W;
            }
            val = s;
        }
        BinaryStdOut.close();
    }
}
```

## Description

The original code had `W = 12` and `L = 4096` hardcoded as constants, meaning every codeword was always 12 bits wide. I made them local variables starting at 9 bits (`L = 512`), and added the widening logic after each dictionary entry gets added.

In `compress()`, once `code` hits `L`, there's no room left at the current width, so `W` goes up by 1 and `L` doubles. The next codeword written uses the new width. `expand()` does the exact same check on `i` after each entry — since both sides widen at the same point (filling the table), the decoder always reads with the right width.

The only other thing to handle was the dictionary array in `expand()`. The original was sized to `L`, but since `L` changes now, I just allocate the full `65536` upfront so there's no issue regardless of how wide things get.

## Runtime and Memory Analysis

**Time complexity: O(n)**

Both `compress()` and `expand()` make a single pass through the input. Each iteration does a TST lookup or array access and a bit read/write — constant or near-constant work per character. The width-widening check is O(1) and fires at most 7 times total (9 → 16).

**Space complexity: O(n) compress, O(2^16) expand**

`compress()` stores the dictionary in a TST that grows with the number of unique substrings seen, bounded by input length. `expand()` always allocates a 65536-entry array regardless of input size, since the maximum codeword space must be reserved upfront.
