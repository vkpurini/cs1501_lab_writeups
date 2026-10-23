# Project 6 Writeup

**Author:** Vivek Purini

---

## Variable-Width LZW Compression

## Solution

```java
public class LZW {
    private static final int R = 256;   // 256 possible input characters (ASCII)
    private static final int MINW = 9;  // start at 9 bits -> 512 codewords
    private static final int MAXW = 16; // stop growing at 16 bits -> 65536 codewords

    public static void compress() {
        // W and L are local so each call starts fresh at 9 bits.
        // If they were static, expand() would inherit compress()'s final values.
        int W = MINW;     // current codeword width in bits
        int L = 1 << W;  // max codewords at this width (2^W). starts at 512.

        String input = BinaryStdIn.readString();

        // dictionary: maps strings -> codeword numbers
        // using a ternary search trie (TST) so it grows dynamically
        TST<Integer> st = new TST<Integer>();

        // pre-load all 256 single characters as codewords 0-255
        for (int i = 0; i < R; i++)
            st.put("" + (char) i, i);

        // codeword 256 (R) is reserved for EOF, so new entries start at 257
        int code = R + 1;

        while (input.length() > 0) {
            // find the longest string in the dictionary that matches the front of input
            String s = st.longestPrefixOf(input);

            // write that string's codeword using the current bit width W
            // this is the only line that actually uses W — everything else manages when W changes
            BinaryStdOut.write(st.get(s), W);

            int t = s.length();

            // add a new dictionary entry: the match s plus the next character after it
            // this is how LZW builds longer patterns over time
            // only add if there's a next char (t < input.length()) and the table isn't full (code < L)
            if (t < input.length() && code < L)
                st.put(input.substring(0, t + 1), code++);

            // if adding that entry just filled the table (code == L), widen by 1 bit
            // the NEXT codeword written will use the new wider W
            // expand() does this same check at the same point, keeping both sides in sync
            if (code == L && W < MAXW) {
                W++;
                L = 1 << W; // double the capacity (2^W)
            }

            // advance past the matched string and repeat
            input = input.substring(t);
        }

        // write EOF codeword (R=256) at the current width so expand knows when to stop
        BinaryStdOut.write(R, W);
        BinaryStdOut.close();
    }

    public static void expand() {
        int W = MINW;         // must start at same width as compress
        int L = 1 << W;       // current codeword limit, grows to match compress
        int MAXL = 1 << MAXW; // 65536 — size the array to the max upfront since L changes

        // dictionary: maps codeword numbers -> strings (reverse of compress's TST)
        // sized to MAXL instead of L because L will grow during expansion
        String[] st = new String[MAXL];
        int i; // next available slot, mirrors compress's `code`

        // pre-load the same 256 single characters at the same indices
        for (i = 0; i < R; i++)
            st[i] = "" + (char) i;
        st[i++] = ""; // slot 256 (R) is the EOF lookahead, leave it empty

        // read the very first codeword to get the loop started
        int codeword = BinaryStdIn.readInt(W);
        if (codeword == R)
            return; // empty input
        String val = st[codeword]; // val = the string for the current codeword

        while (true) {
            BinaryStdOut.write(val); // output the current string

            // read the next codeword at the current width
            // this is where a mismatch with compress's width would break everything
            codeword = BinaryStdIn.readInt(W);
            if (codeword == R)
                break; // EOF marker, done

            String s = st[codeword];

            // special case: the codeword we just read hasn't been added to our table yet
            // this happens when compress used an entry the instant it was created
            // we can still reconstruct it: it must be val + val's first char
            if (i == codeword)
                s = val + val.charAt(0);

            // add a new entry: previous string + first char of current string
            // this mirrors what compress added one step earlier
            if (i < L)
                st[i++] = val + s.charAt(0);

            // widen after the same entry that compress widened after
            // expand is one step behind compress, but both widen when i/code hits L
            // so the next readInt(W) uses the new width, matching compress's next write
            if (i == L && W < MAXW) {
                W++;
                L = 1 << W;
            }

            val = s; // move forward
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
