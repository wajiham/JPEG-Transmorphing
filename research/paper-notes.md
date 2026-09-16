# Paper notes
[IMAGE TRANSMORPHING WITH JPEG] BY LIN YUAN & TOURADJ IBRAHIMI

These are the main concepts I needed to understand before going through the transmorphing method in detail.

---

## Difference Image

The first step is to compare the original image with the processed image and see what actually changed.

A simple way to do that is to calculate the absolute difference between corresponding pixels:

\[
D(x,y) = |I_{original}(x,y) - I_{processed}(x,y)|
\]

For example:

```text
Original:
10  10  10  10
10  50  50  10
10  50  50  10
10  10  10  10

Processed:
10  10  10  10
10  90  92  10
10  88  91  10
10  10  10  10
```

The difference image would be:

```text
0   0   0   0
0  40  42   0
0  38  41   0
0   0   0   0
```

So the difference image tells us **how much each pixel has changed**.

---

## Thresholding

Not every small difference has to be treated as an important modification. A threshold is used to decide whether a pixel difference is large enough to count as changed.

For example, if:

```text
threshold = 20
```

then:

```text
difference > 20  → changed
difference <= 20 → unchanged
```

The threshold therefore controls how sensitive the system is to changes. A low threshold means even small differences are considered important. A high threshold means only larger changes are kept.

---

## Binary Mask

After thresholding, the result can be represented as a binary mask. 
A binary mask is just a map of changed and unchanged regions:

```text
0 = unchanged
1 = changed
```

For the previous example:

```text
0  0  0  0
0  1  1  0
0  1  1  0
0  0  0  0
```

This mask shows where the image has been modified.

The general flow is:

```text
Original image
      +
Processed image
      ↓
Difference image
      ↓
Threshold
      ↓
Binary mask
```

---

## Dilation

The paper does not keep the mask exactly at pixel level. The modified area is expanded so that it matches JPEG block or MCU boundaries. This expansion is called dilation.

For example, a small changed area might look like:

```text
0 0 0 0 0
0 0 1 0 0
0 0 0 0 0
0 0 0 0 0
```

After dilation, the changed region becomes larger:

```text
0 1 1 1 0
0 1 1 1 0
0 1 1 1 0
0 0 0 0 0
```

The idea is not just to enlarge the region randomly. JPEG data is organized in blocks and MCUs, so if a small part of one MCU is changed, the paper treats the corresponding MCU as affected.This makes it easier to preserve and later replace the JPEG data for that region.

---

## Why the Mask is Aligned with MCUs

In a common JPEG using 4:2:0 chroma subsampling, one MCU covers a 16×16 region of the image.

That 16×16 region contains:

```text
4 Y blocks
1 Cb block
1 Cr block
```

where every block is 8×8.

```text
Y:
+--------+--------+
|  8×8   |  8×8   |
+--------+--------+
|  8×8   |  8×8   |
+--------+--------+

Cb:
+--------+
|  8×8   |
+--------+

Cr:
+--------+
|  8×8   |
+--------+
```

The important thing is that the 16×16 size refers to the image area covered by the MCU. The 8×8 blocks are the JPEG blocks inside that area.

There is no conversion from 8×8 to 16×16. Four 8×8 Y blocks together cover one 16×16 region.

---

---

## Threshold Trade-off

The threshold has an important effect on both file size and reconstruction quality.

### Low threshold

```text
low threshold
→ more modified regions
→ more recovery data stored
→ larger file
→ better reconstruction
```

### High threshold

```text
high threshold
→ fewer modified regions
→ less recovery data stored
→ smaller file
→ lower reconstruction quality
```

This is one of the main trade-offs studied in the paper.

---

## Bitrate Overhead

The more modified blocks there are, the more original information needs to be preserved for recovery.

```text
more modified blocks
→ more recovery data
→ higher bitrate / file-size overhead
```

The paper reports that this increase is approximately linear. There can also be some overhead even when very little is stored because the recovery information itself still needs headers and JPEG structure.

---

## Reconstruction Quality

The paper also studies how close the recovered image is to the original. This is measured using PSNR.

---

## MSE

PSNR is based on another metric called Mean Squared Error (MSE). MSE measures the average squared difference between the original and reconstructed pixel values.

For example:

```text
Original:      100  120  130
Reconstructed: 102  118  135
```

Differences:

```text
-2   2   -5
```

Squared:

```text
4   4   25
```

Average:

\[
MSE = \frac{4 + 4 + 25}{3}
\]

\[
MSE = 11
\]

A smaller MSE means the reconstructed image is closer to the original.

---

## PSNR

PSNR stands for Peak Signal-to-Noise Ratio. It is used to measure reconstruction quality.

For an 8-bit image:

\[
PSNR = 10\log_{10}\left(\frac{255^2}{MSE}\right)
\]

The main relationship to remember is:

```text
MSE ↓  → PSNR ↑
MSE ↑  → PSNR ↓
```

So:

```text
higher PSNR = better reconstruction
lower PSNR  = worse reconstruction
```

If the reconstructed image is exactly identical to the original, then `MSE = 0` and PSNR approaches infinity.

Roughly:

```text
20 dB → poor
30 dB → reasonable
40 dB → very good
50 dB → extremely close
```

The paper discusses choosing a threshold that can keep reconstruction around 40 dB while still keeping the additional bitrate relatively low.

---

## Connecting Threshold, File Size, and PSNR

```text
Lower threshold
→ more recovery information
→ larger file
→ lower MSE
→ higher PSNR
```

and:

```text
Higher threshold
→ less recovery information
→ smaller file
→ higher MSE
→ lower PSNR
```

This is the balance the paper is trying to achieve. The goal is not simply to maximize reconstruction quality or minimize file size. It is to find a useful point between the two.

