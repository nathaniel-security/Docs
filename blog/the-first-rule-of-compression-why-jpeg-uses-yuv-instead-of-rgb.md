# The First Rule of Compression: Why JPEG Uses YUV Instead of RGB

While studying custom encoding, decoding, and communication channels, I found myself going down a rabbit hole that initially seemed completely unrelated to cybersecurity:

**JPEG compression.**

At first glance, image compression appears to be a topic for photographers, graphics engineers, or video streaming platforms.

So why should a cybersecurity professional care?

The answer is surprisingly simple.

Think about:

* Building a custom communication channel
* Covert channels
* Malware communication
* Steganography
* Exfiltrating information without drawing attention

One of the first problems you run into is bandwidth.

If your communication channel consumes excessive bandwidth, it becomes noticeable.

The SOC team starts asking questions.

Detection systems begin flagging anomalies.

Network monitoring tools identify unusual traffic patterns.

Defenders often look at metrics such as:

* Average bandwidth consumption
* Traffic volume
* Packet frequency
* Session duration
* Data transfer anomalies

The goal is not simply to communicate.

The goal is to communicate while appearing normal.

The goal is to be discreet.

The goal is to be covert.

This led me to a question:

> How do mature engineering systems decide what information is important and what information can be discarded?

That question eventually led me to studying how JPEG encoding and decoding works.

What I discovered was a lesson that applies far beyond images.

***

## The First Rule of Compression

The first rule of compression is surprisingly simple:

> Throw away what nobody notices.

JPEG is essentially an implementation of this principle.

Instead of trying to preserve every piece of information equally, JPEG identifies the information humans care about and aggressively reduces the information humans barely perceive.

The result is massive storage savings with minimal visible quality loss.

***

## The Obvious Question

A digital image is naturally represented using RGB.

```
Red
Green
Blue
```

Each channel typically consumes 8 bits.

```
R = 8 bits
G = 8 bits
B = 8 bits

Total = 24 bits per pixel
```

However, JPEG does not immediately compress RGB values.

Instead, it converts RGB into YUV (There is a whole calculation of the number which i did not go into)

```
Y = 0.299R + 0.587G + 0.114B
U = (B - Y) x 0.565
V = (R - Y) x 0.713
```

Where:

```
Y = Luminance (Brightness)
U = Blue Color Difference
V = Red Color Difference
```

This naturally raises a question.

**If JPEG converts RGB into YUV, aren't we still storing three values?**

This was the exact question that made me dive deeper into the topic.

```
Y = 8 bits
U = 8 bits
V = 8 bits

Total = 24 bits per pixel
```

The answer is yes.

Raw RGB and raw YUV consume exactly the same amount of storage.

In fact, converting RGB into YUV introduces additional computational overhead.

JPEG now needs to:

* Encode RGB into YUV
* Decode YUV back into RGB during display

So why go through this additional work if the storage remains identical?

The answer lies in human perception.

***

## Human Vision Is Not Balanced

Humans are extremely sensitive to brightness.

Humans are significantly less sensitive to color detail.

Consider the following examples:

* Black text on a white background remains easy to read
* Slight color blur often goes unnoticed
* Slight brightness blur is immediately visible

Our visual system prioritizes brightness information over color information.

JPEG exploits this fact.

Instead of treating all information equally, JPEG separates **Brightness Information**

from **Color Information**

Once these two components are separated, they can be handled differently.

***

## Understanding YUV

YUV separates image information into two categories.

### Y Component

```
Luminance
```

Brightness information.

This is where most image detail exists:

* Edges
* Shapes
* Textures
* Object boundaries

Most of what your eyes perceive as image quality is contained within the Y channel.

### U and V Components

```
Chrominance
```

Color information.

These channels represent color differences rather than actual brightness.

JPEG treats these channels differently because humans are less sensitive to color detail.

***

## What Is Chroma Subsampling?

Once JPEG converts RGB into YUV, it performs a process known as **Chroma Subsampling**.

The word _chroma_ simply means color information.

_Subsampling_ means storing fewer samples than originally existed.

The idea is simple:

* Store brightness at full resolution
* Store color at reduced resolution

Since humans care more about brightness than color, the visual impact is surprisingly small.

***

## 4:4:4 - No Subsampling

No information is removed.

```
Y1 U1 V1
Y2 U2 V2
Y3 U3 V3
Y4 U4 V4
```

Every pixel stores:

* One Y value
* One U value
* One V value

Maximum quality.

Maximum storage.

No compression benefit.

***

## 4:2:2 - Half Horizontal Chroma

```
Y1 U1 V1
Y2

Y3 U2 V2
Y4
```

Two neighboring pixels share the same color information.

Brightness remains unique.

Color becomes shared.

The result is less color information and reduced storage requirements.

***

## 4:2:0 - The Most Common JPEG Format

```
Y1
Y2
Y3
Y4

One U
One V
```

Four pixels now share the same color information.

Each pixel still retains its own brightness value.

Color information is reduced by approximately 75%.

Most JPEG images use this format.

***

## Real Storage Savings

Consider a Full HD image.

```
1920 x 1080
```

Total pixels:

```
1920 x 1080 = 2,073,600 pixels
```

### RGB

```
1920 x 1080 x 3 = 6,220,800 bytes
~6.2 MB
```

### YUV 4:2:0

Brightness:

```
Y 1920 x 1080 = 2,073,600 samples
```

Color:

```
U 960 x 540 = 518,400 samples
```

```
V 960 x 540 = 518,400 samples
```

Total:

```
2,073,600
+
518,400
+
518,400
=
3,110,400 bytes

~3.1 MB
```

Before JPEG compression has even started, the image size has already been reduced by nearly 50%.

***

## The DCT Phase

Up until this point JPEG has only reduced color information.

The image is still not truly compressed.

Now JPEG starts preparing the image for aggressive compression.

The image is divided into small blocks:

```
8 x 8 pixels
```

Each block contains:

```
64 pixels
```

JPEG then applies something called the **Discrete Cosine Transform (DCT)**.

The DCT is closely related to the Fast Fourier Transform (FFT).

However, instead of looking at an image as individual pixels, DCT asks a different question:

> How much detail exists inside this block?

This is where the idea of **spatial frequency** appears.

***

## Understanding Spatial Frequency

Imagine drawing a straight road.

The brightness changes very slowly.

```
Gray
Gray
Gray
Gray
Gray
```

Now imagine drawing a zebra crossing.

```
Black
White
Black
White
Black
```

The brightness changes very rapidly.

These two examples contain different frequencies.

### Low Spatial Frequency

Brightness changes slowly.

```
Gray
Gray
Gray
Gray
Gray
```

Examples:

* Sky
* Walls
* Large objects
* Backgrounds

### High Spatial Frequency

Brightness changes rapidly.

```
Black
White
Black
White
Black
```

Examples:

* Hair
* Grass
* Textures
* Sharp edges

Humans care much more about the large structure of an image than tiny details.

JPEG takes advantage of this.

***

## What DCT Actually Does

DCT converts pixel values into frequency values.

Instead of saying:

```
Pixel 1 = 120
Pixel 2 = 125
Pixel 3 = 122
```

DCT says:

```
This block contains:

80% low-frequency information
15% medium-frequency information
5% high-frequency information
```

This makes it much easier to identify which information is important and which information can be discarded.

Think of DCT as sorting image information by importance.

***

## DC and AC Coefficients

After DCT runs, JPEG produces 64 values.

These values are called coefficients.

### DC Coefficient

The very first coefficient is called the DC coefficient.

```
(0,0)
```

Think of it as:

> The average brightness of the entire 8x8 block.

If an image block mostly contains a blue sky, the DC coefficient will represent the overall brightness of that sky.

It captures the broad structure of the image.

### AC Coefficients

The remaining 63 coefficients are called AC coefficients.

These represent detail.

Examples:

* Edges
* Textures
* Fine patterns
* Small variations

The farther away a coefficient is from the top-left corner, the higher its frequency.

```
Top Left
=
Low Frequency
=
Important

Bottom Right
=
High Frequency
=
Less Important
```

At this stage JPEG has effectively separated:

```
Important image information
```

from

```
Fine image detail
```

***

## Quantization - Where Compression Actually Happens

Everything up until now has been reversible.

DCT does not remove information.

The real compression happens during quantization.

Imagine these values:

```
45
98
23
66
7
```

If perfect accuracy is not important, you might round them:

```
40
100
20
70
10
```

Or simplify them further:

```
4
9
2
6
0
```

Some precision is lost.

However, the numbers become much easier to store.

JPEG applies the same idea to DCT coefficients.

***

## Why Quantization Works

Remember what DCT gave us.

It separated:

```
Important Information
```

from

```
Less Important Information
```

JPEG preserves low-frequency coefficients because humans notice them.

JPEG aggressively reduces high-frequency coefficients because humans barely notice them.

Many coefficients become:

```
0
```

after quantization.

For example:

Before:

```
15  8  3  1  0
12  4  2  1  0
 5  2  1  0  0
```

After:

```
15  8  0  0  0
12  0  0  0  0
 0  0  0  0  0
```

Large amounts of image detail disappear.

Yet visually, the image still looks almost identical.

***

## Lessons For Security Engineers

JPEG is not really an image format.

It is an information prioritization system.

The same principles appear in:

* Covert channels
* Steganography
* Protocol design
* Malware C2
* Detection engineering
* Data exfiltration

The question is always the same:

> What information can be safely ignored?

The engineer uses that answer to compress data.

The attacker uses that answer to hide data.

The defender uses that answer to detect data.

***

## Conclusion

At first glance JPEG's RGB-to-YUV conversion appears pointless.

Both formats initially consume the same storage.

JPEG even introduces additional computational overhead.

However, the conversion unlocks something incredibly powerful:

The ability to treat brightness and color differently.

Once brightness and color are separated, JPEG can:

* Reduce chroma information
* Transform image data into frequencies
* Remove insignificant detail
* Compress efficiently

The entire system is built around a single principle:

> Throw away what nobody notices.

Whether we are compressing images, designing protocols, creating covert channels, building malware communication systems, or engineering efficient data formats, the lesson remains the same.

**Identify the information that matters. Protect it. Everything else is an optimization opportunity.**
