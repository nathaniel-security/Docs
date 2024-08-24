# Why does encoded message have a = or ==

### Base64 Basics

* Converts binary data to text using 64 characters (A-Z, a-z, 0-9, +, /)
* Used for sending binary data through text-only systems (e.g., email)

### How It Works

* Takes 3 bytes (24 bits) of binary data at a time
* Splits these 24 bits into four 6-bit groups
* Each 6-bit group is represented by one base64 character

### The '=' Padding

* Added when the input length is not divisible by 3
* Ensures the output length is always a multiple of 4 characters

### Why Padding is Needed

* Tells the decoder how many meaningful bytes are in the last group
* Prevents decoding errors

### Padding Rules

* No '=' : Input length is divisible by 3
* One '=': Two bytes in the final group (4 real bits in last character)
* Two '==': One byte in the final group (2 real bits in last character)

### Example

* "Man" → "TWFu" (3 bytes, no padding)
* "Ma" → "TWE=" (2 bytes, one '=' padding)
* "M" → "TQ==" (1 byte, two '==' padding)

### Key Point

* The '=' is not part of the encoded data; it's just a marker for the decoder
