# JPEG Background Notes

These are the background concepts I need before going deeper into JPEG transmorphing.

---

## Digital images

A digital image is basically a grid of pixels.

For a grayscale image, each pixel can be thought of as one number. A smaller value is darker and a larger value is brighter. Usually the range is 0 to 255.

```text
0   = black
255 = white
```

So a tiny grayscale image could look like this:

```text
20  25  30
22  28  35
25  32  40
```

A color image usually has three values per pixel instead of one. In RGB, those values tell us how much red, green and blue are present.

```text
(255, 0, 0) = red
(0, 255, 0) = green
(0, 0, 255) = blue
```

This also explains why raw images become large very quickly. For example, a 4000 × 3000 RGB image has 12 million pixels. If each pixel needs 3 bytes, that is around 36 MB before compression. That is where formats like JPEG become useful.

---

## JPEG basics

JPEG is mainly about reducing image size.

The important thing is that JPEG is **lossy**. It does not try to preserve every single piece of image information exactly. Instead, it throws away some information that is usually less noticeable to human eye.

A simplified JPEG flow looks like this:

```text
RGB
↓
YCbCr
↓
chroma subsampling
↓
8×8 blocks
↓
DCT
↓
quantization
↓
encoding
↓
JPEG file
```

At first I thought DCT was the part that causes the loss but that is not really the main issue. DCT mostly changes the way the image data is represented.

The bigger loss happens during quantization, because values are divided and rounded.

So the rough idea is:

- DCT reorganizes the information
- quantization removes precision
- the remaining data is then compressed efficiently

---

## RGB and YCbCr

Images are often represented in RGB, where every pixel has red, green and blue values.

JPEG usually converts this into **YCbCr** before compression.

The channels are:

- **Y**: brightness
- **Cb**: blue-related color information
- **Cr**: red-related color information

The reason JPEG does this is that our eyes are more sensitive to brightness changes than to very fine color changes.

So instead of treating brightness and color equally, JPEG can keep more brightness detail and reduce some color detail.

This becomes important when looking at chroma subsampling and MCUs.

---

## JPEG and 8×8 blocks

JPEG does not process the whole image at once.

It breaks the image into small 8×8 blocks.

```text
+--------+--------+--------+
|  8×8   |  8×8   |  8×8   |
+--------+--------+--------+
|  8×8   |  8×8   |  8×8   |
+--------+--------+--------+
```

Each block has 64 values.

The reason this works well is that pixels inside a small area are often quite similar.

For example, a small part of the sky could look something like:

```text
120 121 121 122
120 120 121 122
119 120 121 121
```

There is a lot of repetition and very little change.

JPEG takes each 8×8 block and runs the DCT on it.

This block structure matters a lot for transmorphing because later we care about which JPEG blocks were actually changed.

---

## DCT

DCT stands for **Discrete Cosine Transform**.

The easiest way for me to think about it is:

> it changes an 8×8 block from pixel values into frequency values.

So instead of describing a block directly using its pixels, JPEG describes how much low-frequency and high-frequency information is inside it.

```text
pixels → DCT → frequency coefficients
```

### What does frequency mean here?

A smooth area changes slowly, so it has more low-frequency information.

Examples:

- sky
- wall
- smooth skin
- blurred background

An area with lots of quick changes has more high-frequency information.

Examples:

- hair
- text
- edges
- grass
- fine texture

After DCT, we still get an 8×8 matrix.

The top-left value is called the **DC coefficient**. It is related to the overall brightness of the block.

The rest are **AC coefficients**, which describe the changes and details inside the block.

Very roughly:

```text
low frequency --------------------> high frequency

+-----------------------------------+
| DC   low   low                    |
| low                               |
|                                   |
|                         high      |
|                              high |
+-----------------------------------+
```

One thing I need to remember: DCT itself is reversible.

If all the coefficients are kept accurately, the inverse DCT can reconstruct the block.

The actual loss mainly comes in the next step, which is quantization.

---

## Quantization

Quantization is where JPEG starts throwing away precision.

Suppose a DCT coefficient is:

```text
61
```

and the quantization value is:

```text
10
```

JPEG does something like:

```text
61 / 10 = 6.1
```

Then it rounds:

```text
6.1 → 6
```

When we reconstruct it later:

```text
6 × 10 = 60
```

The original value was 61, so we no longer have the exact original value. That is why this step is lossy.

Quantization is useful because many small coefficients become zero.

For example:

```text
before: 100, 30, 14, 5, 2, 1
after:   25,  5,  2, 0, 0, 0
```

Those zeros are much easier to compress.

So for me the main distinction is:

```text
DCT          = changes representation
Quantization = loses precision
```

---

## Chroma subsampling

After converting RGB to YCbCr, JPEG can reduce some of the color information. This is called chroma subsampling. The reason it works is that our eyes usually notice brightness detail more than small color differences.

JPEG usually converts an image from RGB into YCbCr:

Y = brightness
Cb = blue-ish color information
Cr = red-ish color information

Our eyes are much more sensitive to changes in brightness than to tiny changes in color. So JPEG keeps more detail for Y but it can reduce the resolution of Cb and Cr.
```
Example: 4 pixels in a row.

For brightness, JPEG might keep all 4 values:

Y:   120  125  130  128
```
For color, instead of storing 4 separate Cb values and 4 separate Cr values, it might store fewer samples and let nearby pixels share them.

So conceptually:
```
Brightness:
Y1  Y2  Y3  Y4

Color:
Cb1     Cb2
Cr1     Cr2
```
That is the main idea of chroma subsampling: fewer color samples, while brightness stays more detailed.

For 4:2:0, it is a 2×2 group of pixels:
```
Pixel 1   Pixel 2
Pixel 3   Pixel 4
```
Each pixel still has its own brightness value:
```
Y1   Y2
Y3   Y4
```
But those 4 pixels can share one Cb value and one Cr value:
```
Cb1 shared by all 4
Cr1 shared by all 4
```
So instead of storing:
```
4 Y + 4 Cb + 4 Cr
```
we store roughly:
```
4 Y + 1 Cb + 1 Cr
```
That saves a lot of space. This is why, in JPEG 4:2:0, a larger image area can be represented by multiple Y blocks but fewer Cb/Cr blocks. And that is what leads to the 16×16 MCU structure the first paper is discussing.

---

## MCU

MCU means **Minimum Coded Unit**.

This confused me at first because JPEG also uses 8×8 blocks. They are related, but they are not the same thing.

An 8×8 block is the block used for DCT.

An MCU is a group of JPEG blocks that are handled together.

With 4:2:0 chroma subsampling, one MCU can cover a 16×16 area of the image (The MCU size depends on the chroma subsampling scheme).

So I need to keep this distinction clear:

```text
DCT block = 8×8
MCU       = group of blocks
```

This becomes important in transmorphing because if a change affects part of an MCU, the method may treat the whole MCU as changed. That is also why masks sometimes need to be aligned to MCU boundaries.

---

## JPEG file structure

A JPEG file is made up of different sections identified by markers.

A very simplified view is:

```text
SOI
APP0 / APP1 / other APP markers
DQT
SOF
DHT
SOS
compressed image data
EOI
```

Some important ones:

- **SOI** — start of image
- **DQT** — quantization tables
- **SOF** — frame information such as image dimensions
- **DHT** — Huffman tables
- **SOS** — start of compressed scan data
- **EOI** — end of image


What matters for now is understanding that a JPEG file has a proper internal structure and that extra application-specific data can also be stored inside it.

---

## APP markers

JPEG has application markers called:

```text
APP0, APP1, APP2 ... APP15
```

These are places where extra application-specific information can be stored. For example, JPEG files often contain metadata such as EXIF information.

A simplified structure could look like:

```text
[SOI]
[APP0]
[APP1]
[custom APP marker]
[DQT]
[SOF]
[DHT]
[SOS]
[image data]
[EOI]
```

The useful part is that a normal JPEG viewer can usually ignore application data it does not understand and still display the image normally.

That is why APP markers are interesting for transmorphing: extra recovery-related data can potentially be stored in the JPEG without stopping regular image viewers from opening it.

---

## JPEG compression pipeline

Putting everything together:

```text
RGB image
↓
convert to YCbCr
↓
reduce some color detail with chroma subsampling
↓
split into 8×8 blocks
↓
apply DCT
↓
get frequency coefficients
↓
quantize the coefficients
↓
encode the remaining data efficiently
↓
store everything in JPEG format
```

The main things I need to remember:

- JPEG does not store raw RGB pixels directly.
- DCT works on 8×8 blocks.
- DCT changes the representation from pixels to frequencies.
- quantization is where precision is lost.
- chroma subsampling reduces color detail.
- MCUs group JPEG blocks together.
- JPEG files are made of markers and sections.
- APP markers can hold extra application-specific data.

