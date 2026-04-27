# Building a Transformer LM

## 2. Byte-Pair Encoding (BPE) Tokenizer

### 2.1 The Unicode Standard

Unicode is a text encoding standard that maps characters to integer code points.
The standard defines 154,998 characters.

In Python, you can use `ord()` function to convert single Unicode character into its integer representation.
The `char()` function converts an integer Unicode code point into a string with the corresponding character.

```python
>>> ord("£")
163
>>> chr(163)
'£'
```

#### Problem (unicode1): Understanding Unicode

a. What Unicode character does `chr(0)` return? 
> It returns null character.

b. How does this character's string representation (__repr__()) differ from its printed representation?
> `repr()` shows `\x00` (the escape sequence), while `print()` outputs the character invisibly, but it is still there with length 1.

c. What happens when this character occurs in text? It may be helpful to play around with the following in your Python interpreter and see if it matches your expectations:
```python
>>> chr(0)
>>> print(chr(0))
>>> "this is a test" + chr(0) + "string"
>>> print("this is a test" + chr(0) + "string")
```
> When using `print()` it produces a human readable, formatted output, whereas raw text just uses `repr()` which shows the literal representation.
> Character when in text is still there with length 1 but it is invisible to the reader.

### Unicode Encodings

Unicode standard defines mapping: characters -> code points (integers)
It is impractical to train tokenizers directly on Unicode codepoints, since the coavulary would be large (~150K) and sparse (since many characters are quite rare).

Instead, we will use Unicode encoding, which converts Unicode character into a sequence of bytes. The unicode standard itself defines three encodings: UTF-8, UTF-16, UTF-32, with UTF-8 being the dominant encoding (more than 98% of all webpages).

By converting our Unicode codepoints into sequence of bytes, we are essentially taking a sequence of codepoints (integers in the range 0 to 154,997)

#### Problem (unicode2): Unicode Encodings

1. What are some reasons to prefer training our tokenizer on UTF-8 encoded bytes, rather than UTF-16 or UTF-32? It may be helpful to compare the output of these encodings for various input strings.
> UTF-8 uses just 1 byte for ASCII characters, and only uses more bytes when needed. Whereas UTF-16 and UTF-32 use atleast 2 and 4 bytes respectively for each character. Additionally, UTF-8 is backward compatible with ASCII, so training data has a compact, familiar representation.

2. Consider the following (incorrect) function, which is intended to decode a UTF-8 byte string into a Unicode string. Why is this function incorrect? Provide an example of an input byte string that yields incorrect results.
    ```python
    def decode_utf8_bytes_to_str_wrong(bytestring: bytes):
        return "".join([bytes([b]).decode("utf-8") for b in bytestring])
    >>> decode_utf8_bytes_to_str_wrong("hello".encode("utf-8"))
    'hello'
    ```
> Example string: "こんにちは". The function does not produce wrong outputs but rther crashes with `UnicodeDecodeError`. That is because `\xe3` is the start byte of a 3-byte UTF-8 sequence, and trying to decode it alone is invalid. The function fails to decode any multi-byte character.

3. Give a two byte sequence that does not decode to any Unicode character(s).
> b"\x80\x80" is invalid vecause `0x80` is a UTF-8 continuation byte, it is only allowed to appear after a start byte.