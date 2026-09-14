# Paper notes

The main idea I took from the paper is that an image can be modified while still keeping enough information about the changed parts to recover the original later.

Very roughly:

```text
original image
↓
processed image
↓
find what changed
↓
keep recovery information for those areas
↓
store it with the JPEG
```

Then recovery works in the opposite direction:

```text
processed JPEG
↓
read the stored recovery information
↓
put the original information back into the changed regions
↓
reconstruct the original
```

